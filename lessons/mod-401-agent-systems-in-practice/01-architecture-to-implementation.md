# Chapter 1 — From Reference Architecture to Production Implementation

An architect hands you a diagram: boxes for a planner, a tool executor, a memory store, and a result synthesizer, with arrows between them and a paragraph of guarantees ("the executor must be idempotent; the planner never calls tools directly"). That diagram is a **contract**, not a build plan. It tells you what each subsystem must promise to its neighbors and what it may assume from them. Everything inside a box — the framework, the data structures, the retry logic, the prompt — is yours to decide and yours to defend.

Your job at this level is to own one of those boxes end to end: implement it so it honors the contract, expose a stable interface so the boxes around it never depend on your internals, and leave it in a state where the next engineer can change it without reading your mind.

## Read the architecture as a contract

Before writing any code, extract three things from the architecture and write them down:

- **The interface.** What does my subsystem accept, and what does it return? Pin this as a typed signature, not prose. If the architecture says "the executor takes a tool call and returns a result," decide the exact shape now: `execute(call: ToolCall) -> ToolResult`, where both types are defined and versioned.
- **The invariants.** What must always be true? "Idempotent on `call.id`," "never raises past the boundary — failures come back as `ToolResult(status=error)`," "no PII in the returned payload." These become test assertions.
- **The assumptions.** What does the architecture let me rely on from upstream? "The planner has already validated the tool name exists." If that assumption is wrong, that is a conversation with the architect *now*, not a defensive `if` you bury later.

```text
          architecture (the contract)
   ┌───────────────────────────────────────┐
   │  planner ──▶ [ EXECUTOR ] ──▶ memory   │   ← boxes + guarantees
   └───────────────────────────────────────┘
                    │  you own this box
                    ▼
   interface:  execute(call: ToolCall) -> ToolResult
   invariants: idempotent on call.id; never raises out
   internals:  framework? retries? cache?  ← your call
```

## Own the box behind a stable interface

The single most valuable thing you produce is the **interface, frozen early**. Once `execute(call) -> ToolResult` is agreed, the planner team and the memory team can build against it while your internals are still a stub. That is the whole point of a subsystem: it lets work happen in parallel and it isolates blast radius. A bug in your retry logic stays in your box.

Concretely, define the boundary as a protocol and make your first commit a working stub:

```python
from typing import Protocol

class ToolExecutor(Protocol):
    def execute(self, call: ToolCall) -> ToolResult: ...

class StubExecutor:
    """Honors the contract, does nothing real. Unblocks neighbors on day one."""
    def execute(self, call: ToolCall) -> ToolResult:
        return ToolResult(call_id=call.id, status="ok", payload={"stub": True})
```

The stub is not throwaway. It is the executable form of the contract, and it becomes the fixture every neighbor tests against. Your real implementation must remain substitutable for it.

## Find the seams before you fill them

A subsystem that honors a contract is necessary but not sufficient; it also has to be *changeable*. The way you get that is by deciding, up front, where the seams are — the points where you can swap an implementation without touching callers. For an executor those seams are usually: the transport to the actual tool (HTTP, MCP, in-process), the retry/timeout policy, and the result cache. Each becomes an injected dependency, not a hard-coded call.

This is where prototypes and production implementations diverge. A prototype calls `requests.post(...)` inline. A production subsystem injects a `Transport` so the same executor runs against a real HTTP tool, an MCP server, or a fake in tests — without changing a line of executor logic. You are not over-engineering; you are honoring the one promise the architecture cannot make for you: that this code will be modified.

## Trace one request end to end before you call it done

The cheapest way to find a contract violation is to follow a single real request through your subsystem and check every promise at the boundary. Did it return the declared type? Did it stay idempotent when you replayed the same `call.id`? Did an upstream tool timeout surface as `status=error` rather than an exception that escaped the box? A senior engineer does this trace deliberately and writes it up as the subsystem's first integration test — the same instinct as the single-request trace you ran in [mod-204's orchestrator-worker build](https://github.com/ai-infra-curriculum/ai-infra-agentic-ai-engineer-learning/tree/main/lessons/mod-204-multi-agent-implementation), now applied to a boundary you must defend to others.

## Key takeaways

- A reference architecture is a contract: it fixes interfaces, invariants, and assumptions, and leaves the internals to you.
- Freeze the interface early and ship a stub that honors it — neighbors build in parallel and your blast radius stays contained.
- Design the seams (transport, retry, cache) as injected dependencies, because the one guarantee the architecture cannot make is that your code will be changed.
- Prove the contract by tracing a single request through every boundary promise, and capture that trace as your first integration test.
