# Chapter 2 — Token Economics and Latency Budgets

Multi-agent systems trade tokens and latency for quality. Every worker you fan out to is more context, more output, and another network round-trip. That trade is often worth it — but only if you can *see the price*. At a senior level you own a budget: a target cost per resolved task and a latency SLA. This chapter is about measuring both, setting explicit budgets, and pulling the levers that keep you inside them.

## Why multi-agent is expensive

A single agent pays for one context window and one line of reasoning. A multi-agent run multiplies that:

- **Fan-out multiplies tokens.** Five workers each reading their instruction, calling tools, and reasoning can burn 5–15× the tokens of one agent on the same task. Anthropic reported their multi-agent research system used roughly 15× the tokens of a chat interaction — justified only because the task value was high.
- **Orchestration adds round-trips.** Decompose → fan out → synthesize is at least three sequential model calls on the critical path, even before workers do their own multi-turn loops.
- **Re-sending context compounds.** Every turn in an agent loop resends the growing transcript. Long-running workers pay for their whole history on every step.

None of this is wrong. It's the cost of breadth and depth. The job is to make the cost *legible* and *bounded*.

## Measure first: cost and latency per task

You cannot budget what you don't measure. Instrument every run to emit, per *resolved task* (not per API call):

- **Total input + output tokens**, broken down by agent role (orchestrator vs. each worker).
- **Cost in dollars**, computed from current per-model token prices — input and output priced separately, since output is typically several times more expensive.
- **Wall-clock latency**, and the **critical-path latency** (the longest dependent chain), which is what the user actually waits for.

```python
import time
from dataclasses import dataclass, field

@dataclass
class RunBudget:
    max_usd: float
    max_latency_s: float
    spent_usd: float = 0.0
    started: float = field(default_factory=time.monotonic)

    def record(self, in_tok: int, out_tok: int, in_price: float, out_price: float):
        self.spent_usd += in_tok * in_price + out_tok * out_price

    def over_budget(self) -> bool:
        elapsed = time.monotonic() - self.started
        return self.spent_usd > self.max_usd or elapsed > self.max_latency_s
```

Tie token counts to spans in your tracing from [mod-402](../mod-402-eval-observability-infra/README.md) so you can ask "which role, on which task type, is eating the budget?" Aggregate metrics hide the one worker role that quietly doubled in cost after a prompt change.

## Set explicit budgets

A budget is a number you commit to and design against. Make two of them per task type:

- **A token/cost budget** — e.g., "≤ $0.08 per resolved support ticket, p95." This caps fan-out and transcript growth.
- **A latency budget** — e.g., "p95 ≤ 8 seconds." This is what sets your per-hop timeouts from Chapter 1.

Then split the latency budget across the critical path: if the SLA is 8 seconds and you have orchestrator → worker → synthesis, you might allot 1.5s + 5s + 1.5s. Each hop's timeout comes straight from its slice. A budget nobody allocated is a budget nobody can defend.

## The levers that keep you in budget

When measurement shows you're over, these are the levers, roughly in order of leverage:

- **Route by difficulty.** Don't send every step to the most expensive model. A cheap, fast model handles decomposition, classification, and easy workers; reserve the frontier model for genuinely hard reasoning. Model routing is usually the single biggest cost lever.
- **Cache aggressively.** Prompt caching makes repeated context (a long system prompt, a shared corpus) far cheaper on subsequent calls. Cache tool results and identical sub-queries so two workers don't pay twice for the same lookup.
- **Bound fan-out and transcript length.** Cap the number of workers (Chapter 1) and trim worker context to what the assignment needs. Distilled returns (workers hand back summaries, not transcripts) keep the orchestrator's context — and cost — small.
- **Stop early.** If the budget tracker says you've hit the cost or latency cap, stop fanning out and synthesize from what you have. An evaluator-optimizer loop should stop when output clears the bar *or* the budget is spent — whichever comes first.
- **Parallelize the critical path.** Latency, unlike cost, shrinks when independent work runs concurrently. Fanning out five workers in parallel costs the same tokens as serial but finishes in one worker's time, not five.

Note the tension: parallelism cuts *latency* but not *cost*, and routing cuts *cost* but a cheaper model may need more turns. You're optimizing two budgets at once; measure both before and after every change.

## Enforce, don't just observe

A budget you only watch is a billing surprise waiting to happen. Wire enforcement into the loop:

```python
async def run_step(step, budget: RunBudget):
    if budget.over_budget():
        raise BudgetExceeded(f"spent=${budget.spent_usd:.3f} elapsed-cap hit")
    usage = await call_model(step)
    budget.record(usage.input_tokens, usage.output_tokens,
                  IN_PRICE[step.model], OUT_PRICE[step.model])
```

Combine a soft cap (degrade: stop fanning out, synthesize now) with a hard cap (abort the run and return a partial result). The hard cap is your circuit breaker against a decomposition bug that tries to spawn 200 workers — the same runaway Chapter 1 bounds from the concurrency side.

## Key takeaways

- Multi-agent buys breadth and depth by spending tokens and round-trips — often 5–15× a single agent; make the price legible.
- Measure tokens, dollars, and **critical-path** latency per *resolved task*, broken down by agent role.
- Commit to explicit per-task-type **cost** and **latency** budgets; split the latency budget across hops to set your timeouts.
- Biggest levers: route by difficulty, cache repeated context, bound fan-out, stop early, parallelize independent work.
- Enforce budgets in the loop with a soft cap (degrade) and a hard cap (abort) — observation alone is a billing surprise waiting to happen.
