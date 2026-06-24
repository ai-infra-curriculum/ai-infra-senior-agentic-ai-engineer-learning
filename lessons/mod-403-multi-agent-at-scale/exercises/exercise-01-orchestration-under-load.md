# exercise-01: Orchestration Under Load

**Estimated effort:** 4 hours

## Objective

Take an orchestrator-worker system that works for a single request and make it survive production traffic. You'll load-test it to find where it collapses, add bounded concurrency and backpressure so it stops collapsing, put timeouts on every hop, and prove it **degrades gracefully** under partial failure instead of cascading. By the end you'll have numbers — throughput, p95 latency, success rate under load — that you can defend.

## Background

This exercise covers material from:

- [Chapter 1 — Orchestration Topologies Under Load](../01-orchestration-under-load.md)
- [Chapter 2 — Token Economics and Latency Budgets](../02-token-and-latency-budgets.md) (for setting timeouts from a latency budget)

Start from an orchestrator-worker system like the one you built in [mod-204 exercise-01](https://github.com/ai-infra-curriculum/ai-infra-agentic-ai-engineer-learning/tree/main/lessons/mod-204-multi-agent-implementation/exercises/exercise-01-orchestrator-worker-build.md). To keep the load test cheap and deterministic, stub the model call with a function that sleeps a configurable, variable amount and occasionally raises — you're testing the *orchestration*, not the model.

## Prerequisites

- An orchestrator-worker system with concurrent fan-out (`asyncio` in Python).
- A way to drive concurrent requests (a load-test loop, `locust`, or `asyncio` tasks).
- A stubbed worker whose latency and failure rate you can configure.

## Tasks

### 1. Establish a baseline and break it

- Wrap the worker call in a stub: `await asyncio.sleep(random.uniform(0.2, 2.0))`, and raise an exception ~5% of the time.
- Drive the system at rising concurrency (1, 10, 50, 200 simultaneous requests, each fanning out to 5 workers). Record throughput, p95 end-to-end latency, success rate, and peak memory.
- Find the point where it collapses: latency runs away, memory grows unbounded, or the event loop saturates. Write down the numbers.

### 2. Bound the concurrency

- Add a process-wide semaphore capping simultaneous worker calls. Derive the cap from a stated "provider rate limit" (pick one, e.g. 20 concurrent calls) with headroom.
- Re-run the load test. The system should now queue instead of collapse: latency rises but stays bounded and memory flattens.

### 3. Apply backpressure

- Add a high-water mark on in-flight requests. Past it, reject new requests immediately with a `429`-style error and a `Retry-After`, rather than accepting them.
- Show that under overload, rejected-fast requests free the system to *finish* accepted ones — overall successful throughput goes up, not down.

### 4. Timeouts on every hop

- Set a latency budget (e.g., p95 ≤ 8s end-to-end). Split it across the path and give each worker call an `asyncio.wait_for` timeout from its slice.
- Confirm a deliberately slow worker is cut off at its timeout and handled as a failure, not left hanging in a semaphore slot.

### 5. Graceful degradation

- With the 5% worker failure rate (plus your timeouts), make the orchestrator synthesize from the workers that *did* return and explicitly label which assignments are missing.
- Show that a request with 2 of 5 workers failing still returns a useful, correctly-labeled partial answer — and that no failure is silently dropped.

## Starter guidance

```python
import asyncio, random, time

WORKER_LIMIT = asyncio.Semaphore(20)        # task 2: derive from provider quota
IN_FLIGHT_MAX = 100                          # task 3: backpressure high-water mark
WORKER_TIMEOUT_S = 5.0                       # task 4: from the latency budget

class Overloaded(Exception): ...

async def worker_stub(assignment_id: str) -> dict:
    await asyncio.sleep(random.uniform(0.2, 2.0))
    if random.random() < 0.05:
        raise RuntimeError(f"worker {assignment_id} failed")
    return {"assignment_id": assignment_id, "answer": "..."}

async def run_worker(assignment_id: str) -> dict:
    async with WORKER_LIMIT:
        return await asyncio.wait_for(worker_stub(assignment_id), WORKER_TIMEOUT_S)

_in_flight = 0

async def handle_request(assignments: list[str]) -> dict:
    global _in_flight
    if _in_flight >= IN_FLIGHT_MAX:
        raise Overloaded("retry-after: 1")          # backpressure: reject fast
    _in_flight += 1
    try:
        settled = await asyncio.gather(*(run_worker(a) for a in assignments),
                                       return_exceptions=True)
        results = [r for r in settled if not isinstance(r, Exception)]
        failed  = [a for a, r in zip(assignments, settled) if isinstance(r, Exception)]
        return {"results": results, "missing": failed}   # graceful degradation
    finally:
        _in_flight -= 1
```

## Acceptance criteria

You can demonstrate, with recorded numbers, that:

- **Without** bounds, the system collapses under load (runaway latency or unbounded memory) — you have the breaking point documented.
- **With** the semaphore, latency stays bounded and memory flattens at the same load.
- Backpressure rejects excess requests fast (`429` + `Retry-After`), and successful throughput under overload is *higher* with it than without.
- Every worker call has a timeout; a slow worker is cut off and counted as failed, not left holding a slot.
- A request with multiple failed/timed-out workers returns a partial answer that explicitly names the missing parts; no failure is silently swallowed.

## Reflection

In `NOTES.md`:

1. What was the breaking point before bounding, and what limited it first — latency, memory, or the provider quota?
2. How did you choose the semaphore size? What happens if it's set too high vs. too low?
3. Why does rejecting requests fast *increase* successful throughput under overload? Tie it back to Little's Law or queue growth.
4. Where did you set each timeout, and how did you split the latency budget across the path?

## Stretch goals

- Replace the flat fan-out with a **hierarchical** topology (orchestrator → 2 sub-orchestrators → workers) and compare how a failing subtree is contained versus the flat version.
- Add **priority load-shedding**: under backpressure, serve interactive requests and defer batch ones; measure the effect on each class's latency.
- Plot throughput and p95 latency vs. offered load for the unbounded and bounded systems on the same axes — the classic "knee" before and after.
