# Chapter 4 — Hardening Inter-Agent Communication and Tools

Multi-agent systems fail at the seams. The reasoning inside an agent is the part everyone focuses on, but at scale the breakage happens *between* components: the message one agent passes to another, the tool call a worker makes to an external API. These are network boundaries — unreliable, slow, and occasionally hostile — wearing the costume of a function call. This chapter is about hardening those seams so a flaky tool or a malformed message degrades a single request instead of taking down the system.

## Treat every seam as a network boundary

Inside a process, calling a function is reliable: it returns or it raises, instantly. Across a seam — agent → agent, agent → tool, agent → model — none of that holds. The call can hang, return garbage, partially succeed, or succeed twice. Senior engineering is assuming all of those will happen and building for them. Two rules anchor everything else: **validate what crosses the seam**, and **never trust the seam to be reliable**.

## Typed contracts validated at the boundary

A worker that hands the orchestrator free-form prose forces the orchestrator to parse, and parsing failures at 3 a.m. are nobody's idea of fun. Every message between agents should be a **typed, validated contract**:

```python
from pydantic import BaseModel, ValidationError

class WorkerResult(BaseModel):
    answer: str
    sources: list[str]
    confidence: float          # 0.0–1.0
    assignment_id: str

def parse_result(raw: str) -> WorkerResult | None:
    try:
        return WorkerResult.model_validate_json(raw)
    except ValidationError as e:
        log.warning("malformed worker result", error=str(e))
        return None            # handle as a failed worker, don't crash
```

Validate at the boundary, fail fast on bad input, and treat a contract violation as a *failed step* (Chapter 1's graceful degradation), not a crash. The same applies to tool I/O: schema the inputs and outputs, reject what doesn't conform, and never feed unvalidated tool output straight back into a model — that's both a reliability and a security boundary.

## Retries with backoff and jitter

Transient failures — a `429`, a `503`, a dropped connection — are the common case across a seam, and most clear up on a retry. But retry naively and you make outages worse: every client retrying in lockstep the instant a service recovers is a **thundering herd** that knocks it back down.

```python
import asyncio, random

async def with_retry(fn, attempts=4, base=0.5, cap=8.0):
    for i in range(attempts):
        try:
            return await fn()
        except TransientError:
            if i == attempts - 1:
                raise
            delay = min(cap, base * 2 ** i) + random.uniform(0, base)  # backoff + jitter
            await asyncio.sleep(delay)
```

The rules: **retry only transient and idempotent calls** (Chapter 3 — retrying a non-idempotent action duplicates it), use **exponential backoff** so you back off as the problem persists, add **jitter** to desynchronize clients, and **cap the attempts** so a dead dependency doesn't retry forever. Honor a `Retry-After` header when the server gives you one.

## Circuit breakers stop the bleeding

Retries help with a blip. They hurt when a dependency is *down* — you spend the whole timeout budget retrying a service that won't answer, dragging every request down with it. A **circuit breaker** detects sustained failure and stops calling the dead dependency entirely:

- **Closed** (normal): calls pass through; count failures.
- **Open** (tripped): after a failure threshold, fail *immediately* without calling — fast failure beats slow timeout, and it gives the dependency room to recover.
- **Half-open** (probing): after a cooldown, let one trial call through; success closes the breaker, failure reopens it.

A breaker on each tool and downstream agent contains a failing dependency to the features that need it, instead of letting it exhaust your concurrency slots (Chapter 1) on doomed calls. Pair it with a **fallback**: when the breaker is open, return a degraded-but-valid result — a cached answer, a cheaper tool, or an honest "this capability is temporarily unavailable."

## Dead-letter the unrecoverable

Some messages and tasks will exhaust every retry and never succeed: a permanently malformed payload, a tool that's been decommissioned, a poison message that crashes whatever consumes it. Looping on those forever is its own outage. Route them to a **dead-letter queue** — a durable side-channel — so the main flow keeps moving and an operator can inspect, fix, and replay them later. A dead-letter path turns "one bad message wedges the pipeline" into "one bad message gets parked for review."

## Securing the seams

Hardening isn't only about reliability; the seams are also the attack surface. At scale:

- **Authenticate and authorize between agents.** A worker shouldn't be able to invoke tools it has no business using; scope each agent's tool access to least privilege.
- **Treat tool output and inter-agent messages as untrusted input.** They can carry prompt-injection payloads. Validate, and don't let one agent's output silently expand another's authority.
- **Rate-limit and quota per agent.** A buggy or compromised agent shouldn't be able to exhaust a shared tool or budget for everyone.

The contract that makes communication reliable — typed, validated, least-privilege — is the same contract that makes it safe.

## Key takeaways

- Multi-agent systems break at the **seams**; treat every agent-to-agent and agent-to-tool call as an unreliable network boundary.
- Make every message a **typed, validated contract**; a contract violation is a failed step, not a crash, and never feed unvalidated tool output back to a model.
- Retry only **transient + idempotent** calls, with exponential **backoff + jitter** and a capped attempt count, to avoid thundering herds.
- Put a **circuit breaker** on each tool and downstream agent so a dead dependency fails fast with a fallback instead of exhausting your concurrency.
- **Dead-letter** unrecoverable messages so one poison payload can't wedge the pipeline — and secure the seams with least-privilege tool access and untrusted-input handling.
