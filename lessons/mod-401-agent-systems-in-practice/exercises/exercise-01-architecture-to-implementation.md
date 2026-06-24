# exercise-01: Architecture to Implementation

**Estimated effort:** 3 hours

## Objective

Take a one-page reference architecture you did not write and turn one of its boxes into a production-ready subsystem: a frozen, typed interface; a stub that honors the contract; a real implementation with injected seams; an ADR for the one decision the architecture left open; and an integration test that traces a single request across every boundary promise. By the end you will have practiced the core senior move — owning a subsystem behind a stable interface so neighbors can build against you in parallel.

## Background

This exercise covers material from:

- [Chapter 1 — From Reference Architecture to Production Implementation](../01-architecture-to-implementation.md)
- [Chapter 2 — Implementation-Level Tradeoffs](../02-implementation-tradeoffs.md)

You will implement the **tool-executor** box from this reference architecture:

```text
planner ──ToolCall──▶ [ EXECUTOR ] ──ToolResult──▶ synthesizer
```

Guarantees the architecture fixes (the contract you must honor):

- The executor accepts a `ToolCall` and returns a `ToolResult`. It **never raises past its boundary** — upstream tool failures come back as `ToolResult(status="error")`.
- It is **idempotent on `call.id`**: replaying the same call returns the same result without re-invoking the tool.
- The planner has already validated that `call.name` refers to a registered tool (an assumption you may rely on — but see Task 5).

Internals — framework, transport, retry policy, cache — are yours to decide.

## Prerequisites

- A typed language with interfaces/protocols (Python 3.11+ with type hints, or TypeScript).
- A test runner (`pytest`, `vitest`, or equivalent) and the ability to inject fakes.
- No live model or network is required; the "tools" can be in-process functions.

## Tasks

### 1. Pin the contract as types

- Define `ToolCall` (`id`, `name`, `args`) and `ToolResult` (`call_id`, `status`, `payload`, `error`) as typed records.
- Define the boundary as a `ToolExecutor` protocol with a single `execute(call) -> ToolResult` method.

### 2. Ship a contract-honoring stub first

- Implement a `StubExecutor` that returns a valid `ToolResult` for any call without doing real work.
- Treat the stub as the executable contract: every later test asserts your real executor is substitutable for it.

### 3. Implement the real executor with injected seams

- Inject a `Transport` (how a tool actually runs), a retry/timeout policy, and an idempotency `Store`.
- Implement: dispatch the call through the transport, retry on transient failure, convert any raised error into `ToolResult(status="error")`, and short-circuit replays via the store.

### 4. Write the ADR for the open decision

- The architecture did not say where idempotency state lives. Decide (in-process map vs. external store) and capture it as a four-line ADR: status, context, decision, consequence.

### 5. Trace one request as an integration test

- Write a test that drives a real call through transport, retry, and store, and asserts every boundary promise: correct return type, idempotent replay, and a tool timeout surfacing as `status="error"` rather than an exception.
- Add one test for the *assumption* in the contract: feed an unregistered tool name and document — in a comment — whether you defend against it or escalate it to the architect, and why.

## Starter guidance

```python
from typing import Protocol
from dataclasses import dataclass

@dataclass(frozen=True)
class ToolCall:
    id: str
    name: str
    args: dict

@dataclass(frozen=True)
class ToolResult:
    call_id: str
    status: str          # "ok" | "error"
    payload: dict | None = None
    error: str | None = None

class Transport(Protocol):
    def run(self, name: str, args: dict) -> dict: ...   # may raise

class Store(Protocol):
    def get(self, call_id: str) -> ToolResult | None: ...
    def put(self, call_id: str, result: ToolResult) -> None: ...

class ToolExecutor(Protocol):
    def execute(self, call: ToolCall) -> ToolResult: ...

class StubExecutor:
    def execute(self, call: ToolCall) -> ToolResult:
        return ToolResult(call_id=call.id, status="ok", payload={"stub": True})

class RealExecutor:
    def __init__(self, transport: Transport, store: Store, max_retries: int = 2):
        self._transport = transport
        self._store = store
        self._max_retries = max_retries

    def execute(self, call: ToolCall) -> ToolResult:
        raise NotImplementedError  # replay-check → retry loop → error-trap → record
```

## Acceptance criteria

You can demonstrate that:

- `ToolCall` and `ToolResult` are typed, and both `StubExecutor` and `RealExecutor` satisfy the `ToolExecutor` protocol.
- A failing or timing-out transport produces `ToolResult(status="error")` — no exception escapes `execute`.
- Replaying the same `call.id` returns the recorded result and does **not** invoke the transport a second time.
- An ADR file records the idempotency-store decision in four lines.
- The integration test traces one request through all three seams and asserts each boundary promise; the unregistered-tool test documents your handling choice.

## Reflection

In `NOTES.md`:

1. Where did you draw the line between "rely on the planner's validation" and "defend at my boundary"? What would change your answer?
2. Your stub and real executor are interchangeable by design. Name one neighbor team that could have started building against the stub on day one, and what they would have been blocked on otherwise.
3. If the architect later changes the contract to "executor may raise `ToolNotFound`," how large is that change in your design, and why?

## Stretch goals

- Add a `CachingExecutor` decorator that wraps any `ToolExecutor` and serves cache hits — without modifying `RealExecutor`. Prove the decorator is substitutable.
- Swap the in-process store for a Redis-backed one behind the same `Store` protocol, and show no executor code changed.
- Add structured logging at the boundary (call id, tool name, status, retry count, latency) and confirm no tool payload leaks into logs.
