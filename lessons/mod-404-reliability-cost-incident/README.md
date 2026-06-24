# mod-404-reliability-cost-incident: Reliability, Cost & Incident Response

**Estimated effort:** 12 hours

Agentic systems fail in ways traditional services don't. A web service that returns HTTP 200 quickly is usually healthy; an agent can return 200, in 800ms, having confidently hallucinated a refund policy, called a tool it shouldn't have, or looped 40 times burning $14 before quitting. Uptime monitoring catches none of that. This module is the SRE-for-agents discipline: you define **quality** SLOs (not just availability), wire **budgets and circuit breakers** into the running system, run **incident response** for the failure modes that are unique to agents, and write **postmortems** that drive durable fixes instead of the same fire next week.

> **Senior framing.** At L40 you own the on-call rotation and the cloud bill, not just the code. This module is about the systems and runbooks that let a team sleep — error budgets that gate releases, cost guards that fail closed, and a blameless postmortem practice that compounds.

## Learning objectives

- Define **SLOs/SLIs for agentic systems** and alert on **quality drift**, not just uptime — measurable proxies for "is the agent still doing its job well."
- Build **cost controls and budgets** into running systems: per-run caps, per-tenant budgets, circuit breakers, and fail-closed guards.
- Run **incident response** for agent-specific failures — bad outputs, runaway loops, tool misuse — with detection, containment, and a clean command structure.
- Write **postmortems** that are blameless, evidence-based, and produce tracked action items that actually land.

## Lecture chapters

1. [SLOs and SLIs for Agentic Systems](01-slos-and-slis-for-agents.md) — quality SLIs, error budgets, and alerting on drift instead of just 500s.
2. [Cost Controls and Budgets](02-cost-controls-and-budgets.md) — per-run/per-tenant budgets, circuit breakers, and where to put the guard so it fails closed.
3. [Incident Response for Agent Failures](03-incident-response-for-agents.md) — runaway loops, tool misuse, and bad outputs; detect, contain, command.
4. [Postmortems and Durable Fixes](04-postmortems-and-durable-fixes.md) — blameless writing, the five whys without the blame, and action items that ship.

## Exercises

Hands-on practice. Reference solutions live in the paired [solutions repo](https://github.com/ai-infra-curriculum/ai-infra-senior-agentic-ai-engineer-solutions).

- [exercise-01: Define agentic SLOs](exercises/exercise-01-agentic-slos.md) — pick SLIs for quality drift, compute them from traces, set targets and an error budget.
- [exercise-02: Cost controls in production](exercises/exercise-02-cost-controls-in-production.md) — build a per-run budget guard and a per-tenant circuit breaker that fail closed.
- [exercise-03: Agent incident response](exercises/exercise-03-agent-incident-response.md) — run a simulated runaway-loop incident end to end and write the postmortem.

## Quiz

- [Module quiz](quizzes/module-quiz.md) — one knowledge check across all four chapters, with answer key.

## Prerequisites

- [mod-402: Evaluation & Observability Infrastructure](../mod-402-eval-observability-infra/README.md) — you need traces, evals, and structured telemetry to compute SLIs and run incidents.
- [mod-403: Multi-Agent Systems at Scale](../mod-403-multi-agent-at-scale/README.md) — runaway loops and cost blowups are worse with multiple agents.
- Working familiarity with an observability backend (OpenTelemetry plus a metrics/trace store) and on-call basics.

See [resources.md](resources.md) for primary references.
