# mod-403-multi-agent-at-scale: Multi-Agent Systems at Scale

**Estimated effort:** 12 hours

You already know how to *build* a multi-agent system: decompose, fan out, synthesize, hand off. This module is about what happens when that system meets production traffic, a finite token budget, an SLA, and the inevitable partial failure. At a senior level, "it works on my laptop" is the starting line. The job is making it hold up when ten thousand requests arrive at once, when a worker hangs for ninety seconds, when a model provider rate-limits you mid-run, and when finance asks why last month's bill tripled.

This module rebuilds the patterns from a scale and reliability lens. You'll engineer orchestration topologies that degrade gracefully under load and partial failure, put real numbers on token economics and latency budgets, make long-running agent workloads durable and resumable, and harden the seams — inter-agent messages and tool calls — where production systems actually break.

> **Numbers over vibes.** A senior engineer defends decisions with budgets and measurements: p95 latency, tokens per resolved task, error-budget burn, recovery time after a crash. Every chapter here gives you a number to own.

## Learning objectives

- **Implement orchestration topologies that hold up under load and partial failure** — bounded concurrency, backpressure, timeouts, and graceful degradation instead of cascading collapse.
- **Manage token economics and latency budgets in production** — measure cost and latency per task, set explicit budgets, and route to keep both in bounds.
- **Apply durable execution and resumption for long-running agent workloads** — checkpoint state so a crash, deploy, or rate-limit resumes instead of restarting from zero.
- **Harden inter-agent communication and tool integrations** — typed contracts, idempotency, retries with backoff, and circuit breakers across the seams between agents and tools.

## Lecture chapters

1. [Orchestration Topologies Under Load](01-orchestration-under-load.md) — bounded fan-out, backpressure, timeouts, and degrading instead of collapsing when traffic and failures stack up.
2. [Token Economics and Latency Budgets](02-token-and-latency-budgets.md) — measuring cost and latency per task, setting budgets, and the routing/caching levers that keep you inside them.
3. [Durable Execution and Resumption](03-durable-execution.md) — checkpointing, deterministic replay, and idempotency so long-running workloads survive crashes and rate-limits.
4. [Hardening Inter-Agent Communication and Tools](04-hardening-communication.md) — typed message contracts, retries with backoff, circuit breakers, and dead-letter handling across every seam.

## Exercises

Hands-on practice. Reference solutions live in the paired [solutions repo](https://github.com/ai-infra-curriculum/ai-infra-senior-agentic-ai-engineer-solutions).

- [exercise-01: Orchestration under load](exercises/exercise-01-orchestration-under-load.md) — load-test an orchestrator-worker system, add bounded concurrency and backpressure, and prove it degrades gracefully.
- [exercise-02: Durable execution in production](exercises/exercise-02-durable-execution-in-production.md) — checkpoint a long-running workflow so a mid-run crash resumes instead of restarting.
- [exercise-03: Token and latency budgets](exercises/exercise-03-token-and-latency-budgets.md) — instrument cost/latency per task, set budgets, and enforce them with routing and early-stop.

## Assessment

- [Quiz: Multi-Agent Systems at Scale](quizzes/quiz-01-multi-agent-at-scale.md) — 10 questions on load, budgets, durability, and hardening.

## Prerequisites

- [mod-204: Multi-Agent Implementation](https://github.com/ai-infra-curriculum/ai-infra-agentic-ai-engineer-learning/tree/main/lessons/mod-204-multi-agent-implementation) — you can build orchestrator-worker, handoffs, and sub-agent isolation by hand.
- [mod-402: Eval & Observability Infrastructure](../mod-402-eval-observability-infra/README.md) — tracing and metrics you'll lean on to measure load and cost.
- Comfort with `async` Python, queues, and at-least-once delivery semantics.

See [resources.md](resources.md) for primary references.
