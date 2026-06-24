# Chapter 1 — Orchestration Topologies Under Load

A multi-agent system that works for one request can collapse at a thousand. The orchestrator-worker pattern from [mod-204](https://github.com/ai-infra-curriculum/ai-infra-agentic-ai-engineer-learning/tree/main/lessons/mod-204-multi-agent-implementation) fans out concurrently — which is exactly what makes it dangerous at scale. Each request can spawn N workers, each worker holds a model connection, and the total in-flight work is the product of request rate and fan-out. Under load that product is what saturates your provider quota, your event loop, and your wallet.

This chapter is about keeping the topology *bounded* and *degrading on purpose* instead of collapsing by accident.

## The two failure shapes

At scale you face two distinct failures, and they need different fixes.

- **Load collapse.** Demand exceeds capacity. Queues grow without limit, latency climbs until every request times out, and memory balloons holding half-finished work. The system isn't down — it's accepting more than it can finish.
- **Partial failure.** One dependency misbehaves: a worker hangs, a tool errors, a provider rate-limits you. Without isolation, that one slow worker stalls the whole fan-out, and one failing dependency takes down requests that didn't even need it.

Bounded concurrency and backpressure address load collapse. Timeouts, isolation, and graceful degradation address partial failure. A production orchestrator needs all four.

## Bound the concurrency

Unbounded `asyncio.gather` over every assignment, for every request, is the classic runaway. Put a ceiling on simultaneous worker calls with a semaphore:

```python
import asyncio

WORKER_LIMIT = asyncio.Semaphore(16)  # max concurrent model calls, process-wide

async def run_worker(assignment: dict) -> dict:
    async with WORKER_LIMIT:
        return await call_model(assignment)
```

The right limit is derived, not guessed: start from your provider's rate limit and per-worker token cost, leave headroom, and confirm with a load test (exercise-01). A semaphore turns "spawn everything and hope" into a queue you control.

## Apply backpressure

Bounding concurrency only slows the *workers*. If requests still arrive faster than you finish them, the backlog moves up to the request queue and you're collapsing one layer higher. Backpressure pushes that signal back to the caller before the queue is unbounded.

- **Reject early.** When the in-flight count hits a high-water mark, return `429 Too Many Requests` with a `Retry-After` rather than accepting work you can't start soon.
- **Bound every queue.** A bounded `asyncio.Queue(maxsize=...)` makes producers wait (or fail fast) instead of buffering forever.
- **Shed load by priority.** Under pressure, serve interactive requests and drop or defer batch ones.

The principle: a system at capacity should say "not now" quickly, not "yes" slowly. A fast rejection is recoverable; a 90-second timeout that still fails is wasted spend and a bad experience.

## Timeouts on every hop

A worker with no timeout is a worker that can hang forever and pin a slot in your semaphore. Bound every model and tool call, and bound the whole fan-out:

```python
async def run_worker(assignment: dict) -> dict:
    async with WORKER_LIMIT:
        return await asyncio.wait_for(call_model(assignment), timeout=30)
```

Set timeouts from your latency budget (Chapter 2), not from "feels long enough." A worker that exceeds its slice is a failed worker — handle it like one.

## Degrade gracefully, don't collapse

Partial failure is the normal case at scale, so the orchestrator must synthesize from whatever returns. Settle every worker, separate successes from failures, and proceed:

```python
settled = await asyncio.gather(*(run_worker(a) for a in assignments),
                               return_exceptions=True)
results = [r for r in settled if not isinstance(r, Exception)]
failed  = [a for a, r in zip(assignments, settled) if isinstance(r, Exception)]
# synthesize from `results`; tell the user which `failed` parts are missing
```

Graceful degradation is a product decision, not just a `try/except`. Decide in advance: is a partial answer with three of five workers acceptable, or must the request fail? Return a smaller-but-correct result, label what's missing, and never silently drop a failure — a missing worker that goes unmentioned is a correctness bug.

## Topology shapes the blast radius

The same patterns assemble into different topologies, and the topology decides how far a failure spreads.

- **Flat fan-out** (one orchestrator, N workers) is simplest; the orchestrator is the single point of failure and the synthesis bottleneck.
- **Hierarchical** (orchestrator → sub-orchestrators → workers) bounds fan-out at each level and contains failures to a subtree, at the cost of more latency hops.
- **Pipeline / staged** (queue between stages) decouples producers from consumers so a slow stage applies backpressure instead of stalling upstream — and it's the natural home for the durable execution in Chapter 3.

Pick the topology for the failure isolation you need, then apply bounded concurrency, backpressure, timeouts, and graceful degradation *within* it.

## Key takeaways

- Two failures, different fixes: bounded concurrency + backpressure for **load collapse**; timeouts + isolation + graceful degradation for **partial failure**.
- Cap simultaneous worker calls with a semaphore sized from your provider quota and token cost — confirmed by a load test, not a guess.
- A system at capacity should reject fast with `429`/`Retry-After`, not accept work it can't finish.
- Every model and tool call gets a timeout drawn from the latency budget; a hung worker is a failed worker.
- Graceful degradation is a deliberate product decision — return partial-but-correct results and always label what's missing.
