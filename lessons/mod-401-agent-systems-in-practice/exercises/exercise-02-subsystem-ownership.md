# exercise-02: Subsystem Ownership

**Estimated effort:** 3 hours

## Objective

Own a subsystem the way a senior engineer is expected to: not just make it work, but make it *operable*. Starting from the tool-executor you built in exercise-01 (or an equivalent), you will add the things that distinguish a subsystem someone can run in production from one that merely passes tests — an explicit failure policy, observability at the boundary, a health signal, and a runbook another engineer can follow at 3 a.m. without you. The deliverable is ownership, not features.

## Background

This exercise covers material from:

- [Chapter 1 — From Reference Architecture to Production Implementation](../01-architecture-to-implementation.md)
- [Chapter 2 — Implementation-Level Tradeoffs](../02-implementation-tradeoffs.md)
- [Chapter 4 — Build-vs-Buy and Complexity Decisions](../04-build-vs-buy-and-complexity.md)

"Owning a subsystem end to end" means you are the person paged when it misbehaves. That role demands artifacts beyond code: a documented failure policy so behavior under stress is intentional, telemetry so an operator can see what happened, and a runbook so they can act without reverse-engineering your design.

## Prerequisites

- The tool-executor from exercise-01, or any agent subsystem with a typed boundary you can instrument.
- A structured-logging facility and a way to emit counters/timings (a metrics client, or stdout in a structured format).
- A test runner that lets you simulate failure (a transport that times out, errors, or is slow).

## Tasks

### 1. Make the failure policy explicit

- Enumerate the failure modes at your boundary: transient transport error, hard tool error, timeout, rate-limit/backpressure, and malformed tool output.
- For each, decide and **document** the behavior: retry (how many, what backoff), fail closed (`status="error"`), or fail open (degraded result). Put this in a `FAILURE_POLICY.md` table, not just in code.

### 2. Instrument the boundary

- Emit, per `execute` call: a structured log line (call id, tool name, status, retry count, latency) and metrics (a success/error counter and a latency timing).
- Enforce one invariant: **no tool payload or argument values leak into logs or metrics** — log identifiers and shapes, not contents. Add a test that asserts a secret-looking arg never appears in emitted logs.

### 3. Add a health signal

- Expose a `health()` that reports whether the subsystem can serve (e.g., the store is reachable, the transport is configured) and a recent error rate over a rolling window.
- Decide what "unhealthy" means for *this* subsystem and write it down — a senior judgment call, not a copied threshold.

### 4. Write the runbook

- Produce `RUNBOOK.md` covering: what this subsystem does in one paragraph, its dependencies, the top three alerts an operator might see, and the first diagnostic step for each. Write it for someone who has never read your code.

### 5. Decide one build-vs-buy question and record it

- You need retry-with-backoff and a metrics client. For each, decide build vs. adopt-a-library, applying the Chapter 4 grid, and capture both as ADRs. At least one must justify *not* hand-rolling something a library already solves.

## Starter guidance

Use this skeleton repo layout and fill the artifacts — the prose deliverables carry as much weight as the code here.

```text
executor/
├── executor.py          # execute() now wrapped with telemetry
├── policy.py            # failure-policy constants (retries, timeouts, backoff)
├── telemetry.py         # log line + metrics emit, payload-scrubbing
├── health.py            # health() + rolling error-rate window
├── FAILURE_POLICY.md    # the table from Task 1
├── RUNBOOK.md           # the operator doc from Task 4
├── adr/
│   ├── 001-retry-library.md
│   └── 002-metrics-client.md
└── tests/
    ├── test_failure_modes.py    # each enumerated mode behaves as documented
    └── test_no_payload_leak.py  # secrets never reach logs
```

Failure-policy table shape for `FAILURE_POLICY.md`:

```text
| Failure mode        | Detection            | Action            | Surfaced as        |
| ------------------- | -------------------- | ----------------- | ------------------ |
| Transient transport | exception, retryable | retry x2, backoff | error after budget |
| Timeout             | deadline exceeded    | fail closed       | status="error"     |
| Malformed output    | schema validation    | fail closed       | status="error"     |
```

## Acceptance criteria

You can demonstrate that:

- Every failure mode in `FAILURE_POLICY.md` has a test proving the subsystem behaves exactly as documented.
- Boundary telemetry emits status, retry count, and latency — and a test proves no argument/payload contents leak into logs.
- `health()` returns a defensible signal, and your definition of "unhealthy" is written down.
- `RUNBOOK.md` lets a reader who has not seen your code take a correct first diagnostic step for each top alert.
- Two ADRs record the retry and metrics build-vs-buy decisions, at least one justifying adopting over hand-rolling.

## Reflection

In `NOTES.md`:

1. For one failure mode you chose to fail *open* (or closed), what was the tradeoff, and who is affected if you chose wrong?
2. Your runbook assumes the reader knows nothing about your code. Where did writing it expose a design choice that was only obvious in your head?
3. Which observability signal would you miss most if it were removed during an incident, and why that one?

## Stretch goals

- Add a circuit breaker: after N consecutive transport failures, fail fast for a cooldown window instead of retrying. Show the breaker's state in `health()`.
- Generate the runbook's "top alerts" section from the same constants in `policy.py`, so doc and behavior cannot drift.
- Run a game-day: have a teammate (or your future self, a week later) follow only the runbook to diagnose a fault you inject, and note where the runbook failed them.
