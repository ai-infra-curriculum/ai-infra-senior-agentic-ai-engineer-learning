# Resources for mod-401-agent-systems-in-practice

Primary references for implementing agent systems from an architecture. This module is about judgment, so the reading leans toward decision-making and maintainability, not just APIs. Verify against current docs — agent tooling moves fast.

## Architecture and ownership

- **Anthropic — Building effective agents** ([anthropic.com/engineering/building-effective-agents](https://www.anthropic.com/engineering/building-effective-agents)) — the canonical pattern taxonomy you are now implementing *from an architecture*, with explicit guidance on not adding complexity until it pays for itself.
- **Anthropic — How we built our multi-agent research system** ([anthropic.com/engineering/multi-agent-research-system](https://www.anthropic.com/engineering/multi-agent-research-system)) — a production subsystem in the wild: interface boundaries, state, and the failure modes that show up only at scale.

## Decision records and tradeoffs

- **Michael Nygard — Documenting Architecture Decisions** ([cognitect.com/blog/2011/11/15/documenting-architecture-decisions](https://www.cognitect.com/blog/2011/11/15/documenting-architecture-decisions)) — the original ADR format used throughout this module. Short, dated, durable.
- **adr.github.io — Architecture Decision Records** ([adr.github.io](https://adr.github.io)) — templates and tooling for keeping ADRs in-repo next to the code they govern.
- **Martin Fowler — Is Design Dead?** and the **bliki** ([martinfowler.com/bliki](https://martinfowler.com/bliki/)) — see *YAGNI*, *Reversible* decisions, and *DesignStaminaHypothesis* for the complexity-cost reasoning behind Chapter 4.

## Refactoring safely

- **Martin Fowler — Refactoring** ([refactoring.com](https://refactoring.com/)) — the catalog and the discipline of behavior-preserving change in small steps.
- **Michael Feathers — Working Effectively with Legacy Code** — characterization tests and finding seams are the core of Chapter 3; this book is where those terms come from.

## Frameworks (the choice you must defend, not default)

- **LangGraph** ([langchain-ai.github.io/langgraph](https://langchain-ai.github.io/langgraph/)) — graph-structured orchestration with explicit, checkpointable state; strong when control flow is graph-shaped and resumable.
- **OpenAI Agents SDK** ([openai.github.io/openai-agents-python](https://openai.github.io/openai-agents-python/)) — handoffs, tool use, and tracing as first-class primitives.
- **Claude Agent SDK** ([code.claude.com/docs/en/agent-sdk/overview](https://code.claude.com/docs/en/agent-sdk/overview)) — sub-agent isolation, tool scoping, and session state for production agents.
- **Model Context Protocol** ([modelcontextprotocol.io](https://modelcontextprotocol.io)) — the standard for the tool boundary you draw in Chapter 2; let it carry the transport seam so your executor stays framework-agnostic.

> Whatever you adopt, wrap it behind your own interface. The framework is an implementation detail of your subsystem — never let it leak into the contract you owe your neighbors.
