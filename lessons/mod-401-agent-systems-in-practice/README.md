# mod-401-agent-systems-in-practice: Implementing Agent Systems from an Architecture

**Estimated effort:** 14 hours

By now you can build the patterns. This module is about what changes when you build them *from someone else's architecture, in production, as the person on the hook for the result.* A reference architecture is a contract, not a blueprint — it fixes the boundaries and the guarantees but leaves a hundred implementation decisions to you: which framework, where state lives, how wide the tool surface gets, what to build versus buy. At senior level, your value is not that you can wire an orchestrator-worker loop; it's that you make those decisions deliberately, document them so the next engineer inherits your reasoning instead of your regrets, and own a subsystem to the point where it is testable, debuggable, and boring to operate.

> **Judgment over mechanics.** The patterns are assumed. This module grades how you *choose* between implementations of those patterns, how you turn a prototype into something a team can maintain, and how you defend a tradeoff to an architect who will second-guess it. Expect to write decision records, not just code.

## Learning objectives

- Turn a reference architecture into a production implementation and own a subsystem end to end — from interface contract to deployment-ready code.
- Make and document implementation-level tradeoffs — framework choice, state management, and tool boundaries — as durable decision records.
- Refactor a prototype agent into a maintainable, testable codebase without changing its observable behavior.
- Drive build-versus-buy and complexity decisions at the subsystem level, and know when *not* to add an abstraction.

## Lecture chapters

1. [From Reference Architecture to Production Implementation](01-architecture-to-implementation.md) — reading an architecture as a contract, owning a subsystem behind a stable interface, and the seams where design meets code.
2. [Implementation-Level Tradeoffs](02-implementation-tradeoffs.md) — framework choice, where state lives, and how wide to draw tool boundaries — decided and documented, not defaulted.
3. [Refactoring a Prototype into a Maintainable Codebase](03-prototype-to-production.md) — pulling a working spike apart into testable seams without breaking behavior.
4. [Build-vs-Buy and Complexity Decisions](04-build-vs-buy-and-complexity.md) — adopt, wrap, or build at the subsystem level, and the cost of every abstraction you add.

## Exercises

Hands-on practice. Reference solutions live in the paired [solutions repo](https://github.com/ai-infra-curriculum/ai-infra-senior-agentic-ai-engineer-solutions).

- [exercise-01: Architecture to implementation](exercises/exercise-01-architecture-to-implementation.md) — take a one-page reference architecture and produce a subsystem with a stable interface, an ADR, and a test seam.
- [exercise-02: Subsystem ownership](exercises/exercise-02-subsystem-ownership.md) — own a tool-execution subsystem end to end: contract, failure policy, observability, and a runbook.
- [exercise-03: Prototype to production refactor](exercises/exercise-03-prototype-to-production-refactor.md) — refactor a single-file prototype agent into a layered, tested codebase with behavior held constant.

## Prerequisites

- [mod-204: Multi-Agent Systems Implementation](https://github.com/ai-infra-curriculum/ai-infra-agentic-ai-engineer-learning/tree/main/lessons/mod-204-multi-agent-implementation) — you implemented these patterns by hand; here you build them from an architecture.
- Comfort designing interfaces and writing tests in a typed language (Python with type hints, or TypeScript).
- Familiarity with at least one agent framework (LangGraph, the OpenAI Agents SDK, or the Claude Agent SDK) so framework-choice tradeoffs are concrete rather than abstract.

See [resources.md](resources.md) for primary references.
