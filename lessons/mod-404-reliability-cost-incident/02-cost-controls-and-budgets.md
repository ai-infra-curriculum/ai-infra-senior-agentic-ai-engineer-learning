# Chapter 2 — Cost Controls and Budgets

Agent cost is unbounded by construction. A normal API call costs what it costs; an agent decides *at runtime* how many model calls, how many tool calls, and how large a context to assemble. A bad prompt, a confused tool, or a hostile user can turn a 3-cent request into a $40 one — and at scale into a five-figure surprise on the monthly invoice. Reliability engineering for agents therefore includes **cost engineering**: budgets and guards built into the running system, not a spreadsheet reviewed at month-end.

## Three layers of control

Think in nested scopes, each with its own limit and its own failure response.

- **Per-run budget.** A hard cap on a single agent invocation: max steps, max tokens, max tool calls, max wall-clock. Protects against one runaway request. This is your last line of defense against the loop incidents in Chapter 3.
- **Per-tenant / per-feature budget.** A rolling spend cap per customer (or per feature) over a window. Protects one noisy tenant from consuming the whole bill and protects you from a single compromised key.
- **Global budget.** A platform-wide rate or spend ceiling, usually enforced at the gateway. The blast-radius backstop when everything else misses.

Each layer answers a different question: *is this request out of control?* / *is this tenant out of control?* / *is the platform out of control?* You need all three because each catches failures the others can't see.

## The per-run guard

The per-run guard lives inside the agent loop and is checked **before each model or tool call**. It tracks cumulative spend and trips on the first limit exceeded. Track tokens, not just steps — a run can stay under a step cap while ballooning context.

```python
from dataclasses import dataclass, field

class BudgetExceeded(Exception):
    """Raised when a run exceeds a hard limit. Fail closed, not open."""

@dataclass
class RunBudget:
    max_steps: int = 12
    max_total_tokens: int = 120_000
    max_usd: float = 0.50
    steps: int = 0
    total_tokens: int = 0
    usd: float = 0.0

    def charge(self, *, tokens: int, usd: float) -> None:
        """Call AFTER each model/tool call to record spend, BEFORE the next."""
        self.steps += 1
        self.total_tokens += tokens
        self.usd += usd

    def check(self) -> None:
        """Call BEFORE each model/tool call. Trips on the first breach."""
        if self.steps >= self.max_steps:
            raise BudgetExceeded(f"step cap {self.max_steps} reached")
        if self.total_tokens >= self.max_total_tokens:
            raise BudgetExceeded(f"token cap {self.max_total_tokens} reached")
        if self.usd >= self.max_usd:
            raise BudgetExceeded(f"usd cap ${self.max_usd:.2f} reached")
```

The agent loop wraps every iteration:

```python
async def run_agent(task, budget: RunBudget):
    state = init_state(task)
    while not state.done:
        budget.check()                          # fail closed before spending
        out = await model_call(state)           # the spend happens here
        budget.charge(tokens=out.tokens, usd=out.usd)
        state = apply(state, out)
    return state.result
```

Two design choices matter. **Fail closed:** when a budget trips, stop and return a graceful, bounded result ("I wasn't able to finish within limits") — never silently continue. **Check before, charge after:** checking before the call means you never overshoot by a full expensive step; charging after means your numbers reflect real usage. Treat the limits as config, not constants, so on-call can tighten them during an incident without a deploy.

## The per-tenant circuit breaker

A per-run cap can't see a tenant that sends 10,000 well-behaved-but-expensive runs in an hour. For that you need a **circuit breaker** keyed by tenant, backed by a shared store (Redis) so it works across instances.

```python
import time

class CircuitOpen(Exception):
    """Tenant breaker is open; reject fast without spending."""

class TenantBreaker:
    def __init__(self, store, *, cap_usd: float, window_s: int = 3600):
        self.store = store          # shared, e.g. Redis
        self.cap_usd = cap_usd
        self.window_s = window_s

    def _key(self, tenant: str) -> str:
        bucket = int(time.time()) // self.window_s
        return f"spend:{tenant}:{bucket}"

    async def guard(self, tenant: str) -> None:
        """Call before admitting a run. Open => reject fast (fail closed)."""
        spent = float(await self.store.get(self._key(tenant)) or 0.0)
        if spent >= self.cap_usd:
            raise CircuitOpen(f"tenant {tenant} over ${self.cap_usd:.2f}/window")

    async def record(self, tenant: str, usd: float) -> None:
        key = self._key(tenant)
        await self.store.incrbyfloat(key, usd)
        await self.store.expire(key, self.window_s * 2)  # auto-clean old buckets
```

The breaker **rejects fast** when open — no model spend on a rejected request, which is exactly what you want when a tenant is the source of an incident. When it trips, emit a metric and (if the tenant is legitimate) page or auto-raise their cap through a reviewed path, rather than silently dropping their traffic.

## Make cost observable, then attributable

You can't control what you can't see, and you can't bill or alert on what you can't attribute. Emit a structured cost record per run — tokens in/out, model, tool calls, USD, **tenant**, **feature** — onto the same telemetry pipeline as your SLIs (mod-402). That single record powers three things: a real-time **cost SLI** ("p99 cost per run"), per-tenant dashboards for capacity planning, and the forensic trail you'll need when Chapter 3's incident is "why did the bill triple overnight." Attribution is what turns "the bill went up" into "feature X for tenant Y, starting at 02:00."

## Key takeaways

- Agent cost is **unbounded at runtime**; control it in the system with nested budgets, not in a month-end review.
- Use three scopes: **per-run** guard (one bad request), **per-tenant** breaker (one bad customer), **global** ceiling (platform backstop).
- The per-run guard **checks before, charges after, and fails closed**; the per-tenant breaker **rejects fast** from a shared store.
- Emit an **attributable** cost record per run (tenant + feature) so cost becomes an SLI and an incident trail, not a surprise invoice.
