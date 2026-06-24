# mod-402-eval-observability-infra: Building Eval & Observability Infrastructure

**Estimated effort:** 14 hours

A junior engineer instruments *their* agent: they capture a trajectory, wire one trace exporter, and write a handful of asserts. That works for one service. It does not scale to a fleet. When ten teams each roll their own eval script, their own span names, and their own definition of "good," you get ten incompatible quality stories and zero ability to compare, gate, or trust any of them. This module is where you stop instrumenting one agent and start building the **infrastructure** that every agent team in your org runs on: a reusable eval harness, a fleet-wide tracing standard, regression gates that block bad deploys, and a paved road that makes the right thing the easy thing.

> **Build the platform, not the probe.** You already know how to evaluate and trace a single agent (that was [mod-205](https://github.com/ai-infra-curriculum/ai-infra-agentic-ai-engineer-learning/tree/main/lessons/mod-205-evaluation-observability) at the junior rung). Your job now is to package those capabilities so a teammate adopts them in an afternoon, not a sprint — and so a regression can never reach production unnoticed.

## Learning objectives

- Build a **reusable evaluation harness** — trajectory, tool-call, and LLM-judge graders behind one stable interface — that any team can point at their agent.
- Stand up **OpenTelemetry-based tracing and dashboards** for agent *fleets*, using the GenAI semantic conventions so every service speaks one vocabulary.
- Define **quality signals and regression gates** that turn eval output into a deploy decision and block regressions in CI.
- Make eval and observability a **paved road** — defaults, templates, and self-service onboarding — that other engineers adopt because it is easier than not.

## Lecture chapters

1. [Building a Reusable Eval Harness](01-reusable-eval-harness.md) — graders behind a stable interface, datasets as artifacts, and a harness a whole team can share.
2. [Tracing and Dashboards for Agent Fleets](02-tracing-and-dashboards.md) — OTel GenAI conventions, a shared resource model, and dashboards that answer fleet-level questions.
3. [Quality Signals and Regression Gates](03-quality-signals-and-gates.md) — turning eval scores into signals, setting thresholds, and gating deploys without flaking the build.
4. [Making Eval and Observability a Paved Road](04-paved-road-adoption.md) — defaults, templates, golden paths, and the adoption metrics that prove it worked.

## Exercises

Hands-on practice. Reference solutions live in the paired [solutions repo](https://github.com/ai-infra-curriculum/ai-infra-senior-agentic-ai-engineer-solutions).

- [exercise-01: Build a reusable eval harness](exercises/exercise-01-reusable-eval-harness.md) — package trajectory, tool-call, and judge graders behind one interface two teams can share.
- [exercise-02: Tracing and dashboards](exercises/exercise-02-tracing-and-dashboards.md) — instrument with OTel GenAI conventions and stand up fleet dashboards.
- [exercise-03: Regression gates](exercises/exercise-03-regression-gates.md) — wire eval into CI with thresholds that block a real regression.

## Quiz

- [Knowledge check](quizzes/README.md) — graders, OTel conventions, gates, and adoption.

## Prerequisites

- [mod-401: Agent Systems in Practice](../mod-401-agent-systems-in-practice/README.md) — you own a subsystem you can instrument.
- Junior-rung eval and tracing: trajectory/tool-call grading and single-agent OTel tracing (mod-205 in the Agentic AI Engineer track).
- Comfort with `async` Python, `pytest`, and a CI system (GitHub Actions or equivalent).

See [resources.md](resources.md) for primary references.
