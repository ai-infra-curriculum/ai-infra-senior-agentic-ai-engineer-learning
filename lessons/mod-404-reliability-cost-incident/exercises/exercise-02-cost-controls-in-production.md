# exercise-02: Cost Controls in Production

**Estimated effort:** 3 hours

## Objective

Build the cost-control layer for an agent: a per-run budget guard that bounds a single invocation and a per-tenant circuit breaker that bounds a noisy customer — both **failing closed**. You'll wire them into an agent loop, emit attributable cost records, and prove that a deliberately runaway run and a deliberately overspending tenant are both stopped without taking down everyone else.

## Background

This exercise covers material from:

- [Chapter 2 — Cost Controls and Budgets](../02-cost-controls-and-budgets.md)

Use the agent loop from earlier modules. You can stub the model/tool calls with a function that returns a fixed token count and USD cost so the exercise is deterministic and free to run.

## Prerequisites

- An agent loop you can wrap (real or stubbed model/tool calls returning `tokens` and `usd`).
- A shared key-value store for the breaker — Redis, or an in-memory stub that mimics `get` / `incrbyfloat` / `expire`.
- The telemetry sink from [mod-402](../../mod-402-eval-observability-infra/README.md) (or stdout JSON lines) for cost records.

## Tasks

### 1. Per-run budget guard

- Implement a `RunBudget` tracking `steps`, `total_tokens`, and `usd` with hard caps.
- Check the budget **before** each model/tool call and charge it **after**, so you never overshoot by a full expensive step.
- On breach, **fail closed**: stop the loop and return a graceful bounded result, never silently continue.
- Make the caps **config**, not constants, so they can be tightened during an incident without a deploy.

### 2. Per-tenant circuit breaker

- Implement a `TenantBreaker` keyed by tenant over a rolling window, backed by the shared store.
- Before admitting a run, the breaker **rejects fast** when the tenant is over cap (no model spend on a rejected request).
- Record spend after each run; expire old buckets so the store doesn't grow unbounded.

### 3. Wire them into the loop

- Admit a run only after the breaker check; run the loop under the per-run guard; record spend to the breaker on completion.
- Confirm a rejected run does **zero** model spend and a budget-tripped run stops at the cap.

### 4. Attributable cost records

- Emit one structured cost record per run: `tokens_in/out`, `model`, `tool_calls`, `usd`, `tenant`, `feature`, `outcome` (`ok` / `budget_exceeded` / `rejected`).
- Show you can aggregate these into a p99-cost-per-run SLI and a per-tenant spend view.

## Starter guidance

```python
from dataclasses import dataclass

class BudgetExceeded(Exception): ...
class CircuitOpen(Exception): ...

@dataclass
class RunBudget:
    max_steps: int = 12
    max_total_tokens: int = 120_000
    max_usd: float = 0.50
    steps: int = 0
    total_tokens: int = 0
    usd: float = 0.0

    def check(self) -> None:        # call BEFORE each call; fail closed
        if self.steps >= self.max_steps:
            raise BudgetExceeded("step cap")
        if self.total_tokens >= self.max_total_tokens:
            raise BudgetExceeded("token cap")
        if self.usd >= self.max_usd:
            raise BudgetExceeded("usd cap")

    def charge(self, *, tokens: int, usd: float) -> None:  # call AFTER each call
        self.steps += 1
        self.total_tokens += tokens
        self.usd += usd

async def run_with_budget(task, budget: RunBudget, emit) -> dict:
    state, outcome = init_state(task), "ok"
    try:
        while not state.done:
            budget.check()
            out = await model_call(state)            # stub returns tokens, usd
            budget.charge(tokens=out.tokens, usd=out.usd)
            state = apply(state, out)
    except BudgetExceeded:
        outcome = "budget_exceeded"                  # graceful bounded result
    emit({"usd": budget.usd, "steps": budget.steps, "outcome": outcome})
    return {"result": getattr(state, "result", None), "outcome": outcome}
```

You do **not** need SLO alerting (exercise-01) or a full postmortem (exercise-03) here.

## Acceptance criteria

You can demonstrate that:

- A deliberately runaway run stops at the per-run cap and returns a graceful bounded result — it never loops unbounded and never silently continues past the cap.
- A tenant pushed over its window cap is **rejected fast** with zero model spend, while other tenants are unaffected.
- Budget caps and tenant caps are config-driven and can be tightened at runtime without a code change.
- Every run emits an attributable cost record, and you can produce a p99-cost-per-run number and a per-tenant spend view from those records.

## Reflection

In `NOTES.md`:

1. Why check the budget *before* and charge *after*? Show the overshoot you'd get if you reversed it.
2. What is the failure mode if the breaker's shared store is unavailable? Does your code fail open or closed, and is that the right choice for cost?
3. Which cap would you tighten first during a runaway-loop incident, and what's the user-facing cost of doing so?

## Stretch goals

- Add a **global** rate/spend ceiling at the admission point as a platform backstop.
- Make the breaker **half-open**: after the window, admit a trickle of probe runs before fully closing, to avoid hammering a still-broken tenant.
- Add a graceful **degrade** path (smaller model / canned response) instead of a hard reject when a tenant is over budget, and compare user impact.
