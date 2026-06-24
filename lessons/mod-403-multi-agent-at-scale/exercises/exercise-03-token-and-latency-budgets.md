# exercise-03: Token and Latency Budgets

**Estimated effort:** 3 hours

## Objective

Put real numbers on what your multi-agent system costs and how long it takes, then enforce a budget. You'll instrument a run to emit tokens, dollars, and critical-path latency *per resolved task* broken down by agent role; set explicit cost and latency budgets; and wire enforcement — routing, early-stop, a soft cap, and a hard cap — that keeps the system inside them. The deliverable is a budget you can defend with measurements, not vibes.

## Background

This exercise covers material from:

- [Chapter 2 — Token Economics and Latency Budgets](../02-token-and-latency-budgets.md)
- [Chapter 1 — Orchestration Topologies Under Load](../01-orchestration-under-load.md) (the hard cap is the cost-side twin of bounded fan-out)

Use a real model provider if you have a small spend cap, or stub the model call to return a usage object with plausible token counts and latency so the budgeting logic is fully testable offline.

## Prerequisites

- A multi-agent run (orchestrator + workers + synthesis) from a prior exercise.
- Current per-model token prices (input and output priced separately) for whatever models you route between.
- Tracing/metrics from [mod-402](../mod-402-eval-observability-infra/README.md), or simple structured logging.

## Tasks

### 1. Instrument cost and latency per task

- For one resolved task, capture: input and output tokens **per agent role** (orchestrator, each worker, synthesizer), dollars (input × input-price + output × output-price, per model), wall-clock latency, and **critical-path** latency (the longest dependent chain — what the user waits for).
- Emit these as structured records tied to spans, so you can later group by role and task type.

### 2. Set explicit budgets

- Pick a task type and commit to two budgets: a cost budget (e.g., ≤ $0.08 per resolved task, p95) and a latency budget (e.g., p95 ≤ 8s).
- Split the latency budget across the critical path (orchestrator slice + worker slice + synthesis slice). These slices set your per-hop timeouts.

### 3. Route by difficulty

- Send cheap steps (decomposition, classification, easy workers) to a small/fast model and reserve the expensive model for genuinely hard reasoning.
- Measure cost and latency before and after routing on the same task set. Report the delta for *both* budgets — note where a cheaper model needed more turns.

### 4. Cache repeated context

- Identify context that repeats (a long shared system prompt, a corpus several workers read) and cache it. Cache identical sub-queries / tool results so two workers don't pay twice.
- Report the token/cost reduction from caching on a run where workers share context.

### 5. Enforce the budget

- Track spend and elapsed time during the run. Add a **soft cap**: when the cost or latency budget is hit, stop fanning out and synthesize from what you have. Add a **hard cap**: abort the run and return a partial result if spend blows past a ceiling (your defense against a decomposition bug spawning 200 workers).
- Show both firing: a run that gracefully stops early at the soft cap, and a runaway run that's aborted by the hard cap.

## Starter guidance

```python
import time
from dataclasses import dataclass, field

IN_PRICE  = {"small": 0.20 / 1_000_000, "large": 3.00 / 1_000_000}   # $/token, illustrative
OUT_PRICE = {"small": 0.80 / 1_000_000, "large": 15.0 / 1_000_000}

@dataclass
class RunBudget:
    max_usd: float
    max_latency_s: float
    hard_usd: float                       # abort ceiling, > max_usd
    spent_usd: float = 0.0
    started: float = field(default_factory=time.monotonic)

    def record(self, model: str, in_tok: int, out_tok: int):
        self.spent_usd += in_tok * IN_PRICE[model] + out_tok * OUT_PRICE[model]

    @property
    def elapsed(self) -> float:
        return time.monotonic() - self.started

    def soft_exceeded(self) -> bool:
        return self.spent_usd > self.max_usd or self.elapsed > self.max_latency_s

    def hard_exceeded(self) -> bool:
        return self.spent_usd > self.hard_usd

def route(step) -> str:
    return "large" if step.is_hard else "small"      # task 3: route by difficulty

async def run_step(step, budget: RunBudget):
    if budget.hard_exceeded():
        raise BudgetExceeded(f"hard cap: spent=${budget.spent_usd:.3f}")
    model = route(step)
    usage = await call_model(step, model=model)
    budget.record(model, usage.input_tokens, usage.output_tokens)
    return usage
```

## Acceptance criteria

You can demonstrate, with recorded numbers, that:

- A run emits tokens, dollars, and **critical-path** latency per resolved task, broken down **by agent role**.
- You have committed cost and latency budgets, with the latency budget split across the critical path into per-hop timeouts.
- Difficulty-based routing reduces cost on the task set, reported against **both** budgets (including any latency cost of extra turns on the cheaper model).
- Caching shared context measurably reduces tokens/cost on a run where workers share context.
- The **soft cap** stops a run early and synthesizes a partial result; the **hard cap** aborts a runaway run — both demonstrated.

## Reflection

In `NOTES.md`:

1. Which agent role dominated cost, and which dominated latency? Were they the same role?
2. What did routing save, and what did it cost? Did the cheaper model ever need enough extra turns to erase the savings?
3. Cost and latency pull in different directions (parallelism cuts latency but not cost; routing cuts cost but may add turns). Where did you have to trade one for the other?
4. Where would you set the soft vs. hard cap in a real product, and what does the user see when each fires?

## Stretch goals

- Add an **evaluator-optimizer loop** that stops when output clears a quality bar *or* the budget is spent — whichever comes first — and report how often each stopping condition wins.
- Build a small **cost dashboard**: cost and p95 latency per task type over time, so a prompt change that doubles a worker's cost is visible immediately.
- Make routing **adaptive**: start cheap, and escalate a step to the expensive model only when the cheap model's confidence (or a validator) says it's not good enough; measure the cost/quality trade.
