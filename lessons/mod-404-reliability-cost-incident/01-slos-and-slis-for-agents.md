# Chapter 1 — SLOs and SLIs for Agentic Systems

The Google SRE model is built on three nested terms. An **SLI** (Service Level Indicator) is a measured number — a ratio of good events to total events. An **SLO** (Service Level Objective) is a target for that number over a window — "99% of requests succeed over 28 days." An **error budget** is what's left over: at a 99% SLO you may "spend" 1% of events on failure before you must stop shipping risk and fix reliability. That machinery is sound. The trap for agentic systems is choosing the *wrong indicators*.

## Why uptime is a lie for agents

A classic availability SLI counts an event as "good" if it returned non-error within a latency bound. For an agent, that definition passes a response that is fast, well-formed, and **wrong**. The dangerous failures are silent: a hallucinated citation, a tool call that mutated the wrong record, an answer that quietly degraded after a prompt change. None of them throw a 500. If your only SLO is availability, your dashboard stays green while the product rots.

So you need SLIs that proxy *quality*, not just *liveness*. The hard part is that quality is expensive and slow to measure (often an LLM-as-judge or a human label), while SLIs must be cheap and continuous. The job of this chapter is to bridge that gap.

## Picking agentic SLIs

Good SLIs for agents fall into four families. Pick a small number — three to five — that map to what "working" means for *your* agent.

- **Task success rate.** The fraction of runs that achieved the user's goal. Measured by an eval (LLM-judge or rule) over a sample of production traces, or by a downstream signal (the user didn't re-ask, the ticket closed).
- **Grounding / faithfulness.** For RAG or tool-using agents: the fraction of claims supported by retrieved context. A drop here is the leading indicator of a hallucination incident.
- **Tool-call validity.** The fraction of tool calls that were well-formed *and* appropriate (right tool, allowed arguments). Cheap to compute from traces and a strong early-warning signal for tool misuse (Chapter 3).
- **Loop / efficiency health.** The fraction of runs that finished within a step and token budget. A rising tail here precedes runaway-loop incidents (Chapter 3) and cost blowups (Chapter 2).

Each is a ratio of *good events / valid events*, exactly like a classic SLI, so the same error-budget math applies.

## Computing an SLI from traces

You instrumented runs in mod-402. An SLI is a windowed aggregation over those traces. Compute it the same way for the live alert and the SLO report so they never disagree.

```python
from dataclasses import dataclass

@dataclass
class RunRecord:
    grounded_claims: int      # claims supported by retrieved context
    total_claims: int         # claims the agent made
    steps: int                # agent loop iterations
    judged_success: bool | None  # eval label, may be unsampled (None)

def grounding_sli(runs: list[RunRecord]) -> float:
    """Fraction of claims that were grounded. The faithfulness SLI."""
    good = sum(r.grounded_claims for r in runs)
    total = sum(r.total_claims for r in runs)
    return good / total if total else 1.0  # no claims => vacuously grounded

def task_success_sli(runs: list[RunRecord]) -> float:
    """Computed over the JUDGED subset only — don't dilute with unlabeled runs."""
    judged = [r for r in runs if r.judged_success is not None]
    if not judged:
        return float("nan")  # not enough signal; alerting must handle NaN
    return sum(r.judged_success for r in judged) / len(judged)
```

Two non-obvious rules live in that code. First, **measure quality over the sampled, judged subset** — you can't afford to judge every run, so success-rate SLIs carry sampling error; size the sample so the confidence interval is tighter than your alert threshold. Second, **handle the empty/NaN case explicitly** so a quiet hour doesn't page someone or, worse, silently read as 100%.

## Alerting on drift, not just on a threshold

A static threshold ("page if grounding < 0.90") is a blunt instrument: it fires late on a slow slide and never on a regression that lands at 0.91 when you used to run at 0.99. Two patterns serve agents better.

- **Burn-rate alerts (from the SRE workbook).** Alert on how fast you are consuming the error budget. A fast burn (e.g., 2% of a 30-day budget in 1 hour) pages now; a slow burn (10% in 3 days) opens a ticket. This catches both the cliff and the slide with one mechanism.
- **Change-point / baseline alerts.** Compare the current window's SLI against a trailing baseline and alert on a statistically significant drop, even if it's still above the absolute SLO. This is what catches "the new prompt quietly cost us 8 points of faithfulness."

Pair them: burn-rate protects the budget; baseline-drift catches regressions the budget is too generous to notice.

## Quality SLIs gate releases

The payoff of an error budget is policy, not a dashboard. When the budget is healthy, ship freely. When it's exhausted — too many low-grounding runs this month — the policy is to **stop shipping prompt/model/tool changes and spend the next cycle on reliability**. For agents this is sharper than for normal services, because the most common cause of a blown quality budget *is* a recent prompt or model change. The error budget turns "the new prompt feels worse" into a number that gates the next deploy.

## Key takeaways

- Availability SLIs pass fast, confident, wrong answers — agents need **quality** SLIs (success, grounding, tool-call validity, loop health).
- An SLI is still `good / valid` events; compute quality SLIs over the **judged sample** and handle the empty case explicitly.
- Alert on **burn rate** (budget protection) *and* **baseline drift** (regression detection), not a single static threshold.
- The error budget is a **release gate**: a blown quality budget stops prompt/model/tool changes until reliability is restored.
