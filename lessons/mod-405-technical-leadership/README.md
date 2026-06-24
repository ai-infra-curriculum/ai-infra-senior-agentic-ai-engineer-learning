# mod-405-technical-leadership: Technical Leadership for Agentic Teams

**Estimated effort:** 10 hours

At L40 you stop being the person who writes the cleverest agent loop and become the person who makes a *team* of engineers ship agentic systems that are safe, reliable, and on-schedule. The skills change shape. A multi-agent PR that looks fine in isolation can be a production incident waiting for a prompt-injected email. An architect's elegant design dies if the team can't execute it. A six-month "agentic transformation" with no sequencing burns budget and trust before it ships anything. This module is about the judgment, standards, and communication that turn good individual engineers into a team that ships agentic software well.

> **Leadership is leverage, not heroics.** Your job is not to review every line yourself or write every prompt. It is to set the standards, paved roads, and review bars that let the team make safe decisions *without you in the room* — and to scope work so they're never set up to fail.

## Learning objectives

- **Lead code review** for agent systems, with a focus on the safety and reliability failure modes that generic review misses — tool authority, prompt-injection surface, loop bounds, and non-determinism.
- **Mentor engineers and set implementation standards** — paved roads, golden-path templates, and the standards that make the right thing the easy thing.
- **Translate between the architect's design and the team's execution** — turning a design doc into buildable, sequenced work without losing the design's intent or the team's reality.
- **Scope and sequence delivery** of an agentic initiative — slicing a large, uncertain effort into a sequence that ships value early and de-risks the unknowns first.

## Lecture chapters

1. [Leading Code Review for Agent Systems](01-agent-code-review.md) — what generic review misses about tool authority, injection surface, loop bounds, and non-determinism, and a concrete agent-PR review checklist.
2. [Mentoring and Paved Roads](02-mentoring-and-paved-roads.md) — setting standards by making the safe path the easy path, golden-path templates, and mentoring that scales your judgment.
3. [Translating Design to Execution](03-design-to-execution.md) — turning an architect's design into buildable work, preserving intent, surfacing the questions a design doc leaves open.
4. [Scoping and Sequencing Delivery](04-scoping-and-sequencing.md) — slicing an agentic initiative so it ships value early, de-risks the unknowns first, and survives contact with reality.

## Exercises

Hands-on practice. Reference solutions live in the paired [solutions repo](https://github.com/ai-infra-curriculum/ai-infra-senior-agentic-ai-engineer-solutions).

- [exercise-01: Agent code review standards](exercises/exercise-01-agent-code-review-standards.md) — apply a safety-and-reliability review rubric to a real agent PR and write the review.
- [exercise-02: Paved roads and standards](exercises/exercise-02-paved-roads-and-standards.md) — write a paved-road RFC and golden-path template the team will actually adopt.
- [exercise-03: Scoping and sequencing](exercises/exercise-03-scoping-and-sequencing.md) — turn a vague agentic initiative into a sequenced delivery plan.

## Prerequisites

- [mod-401: Agent Systems in Practice](../mod-401-agent-systems-in-practice/README.md) — you should have shipped agent systems yourself before you lead a team building them.
- [mod-403: Multi-Agent at Scale](../mod-403-multi-agent-at-scale/README.md) — the failure modes you'll be reviewing for.
- [mod-404: Reliability, Cost & Incident Response](../mod-404-reliability-cost-incident/README.md) — the production stakes that make these standards matter.

See [resources.md](resources.md) for primary references.
