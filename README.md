# AI Engineering · Senior Agentic AI Engineer — Learning Repository

<!-- aicg:site-banner -->
> 🎓 Part of the free, open-source **AI Career Curriculum** ecosystem — [Infrastructure](https://github.com/ai-infra-curriculum) · [ML Engineering](https://github.com/ml-engineering-curriculum) · [AI Engineering](https://github.com/ai-engineering-curriculum) · [Governance](https://github.com/ai-governance-curriculum). Live cohorts &amp; team programs: **[ai-infra-curriculum.github.io](https://ai-infra-curriculum.github.io/)**.
<!-- /aicg:site-banner -->

The **lead/staff build rung (L40)** of the Agentic AI track. You already know how to build agent
patterns by hand — this track is about owning them in production: leading the implementation of
multi-agent subsystems, building the shared eval and observability infrastructure a whole fleet
runs on, owning reliability and the cloud bill, and mentoring the engineers who ship alongside you.

> **Status**: ✅ Curriculum complete — modules, lecture chapters, exercises, and quizzes authored.
> AI-assisted content under ongoing human review. Capstone project specs land on the next
> autonomous content cycle.

---

## 🎯 Overview

This repository is the **Senior Agentic AI Engineer** stage of the AI Infrastructure Curriculum.
It sits between the [Agentic AI Engineer](https://github.com/ai-engineering-curriculum/agentic-ai-engineer-learning)
rung (L30, build) and the Agentic Systems Architect rung (L48, design). Build fundamentals are
owned by the L30 track and linked, not re-taught; this track adds **depth, production scale, and
technical leadership**.

At L40 the job changes shape. You stop instrumenting *your* agent and start building the
infrastructure every agent team runs on. You stop making "it works on my laptop" the finish line
and start defending decisions with p95 latency, tokens per resolved task, and error-budget burn.
You stop reviewing every line yourself and start setting the standards and paved roads that let a
team make safe decisions without you in the room.

### What you'll master

- **Eval & observability infrastructure** — a reusable grading harness (trajectory, tool-call,
  LLM-judge), fleet-wide OpenTelemetry tracing on the GenAI semantic conventions, and regression
  gates that block bad deploys in CI.
- **Multi-agent systems at scale** — orchestration topologies that degrade gracefully under load,
  token and latency budgets you can defend with numbers, and durable execution that survives
  crashes, deploys, and rate-limits.
- **Reliability, cost & incident response** — quality SLOs (not just uptime), cost controls that
  fail closed, agent-specific incident response, and blameless postmortems that drive durable fixes.
- **Technical leadership** — leading code review for agent-specific safety and reliability failure
  modes, mentoring through paved roads, translating an architect's design into sequenced work, and
  scoping an agentic initiative so it ships value early.

---

## 📚 Modules

| Module | Topic | Hours | Exercises | Quiz |
|--------|-------|-------|-----------|------|
| [mod-401](./lessons/mod-401-agent-systems-in-practice/README.md) | Implementing Agent Systems from an Architecture | 14h | 3 | ✅ |
| [mod-402](./lessons/mod-402-eval-observability-infra/README.md) | Building Eval & Observability Infrastructure | 14h | 3 | ✅ |
| [mod-403](./lessons/mod-403-multi-agent-at-scale/README.md) | Multi-Agent Systems at Scale | 12h | 3 | ✅ |
| [mod-404](./lessons/mod-404-reliability-cost-incident/README.md) | Reliability, Cost & Incident Response | 12h | 3 | ✅ |
| [mod-405](./lessons/mod-405-technical-leadership/README.md) | Technical Leadership for Agentic Teams | 10h | 3 | ✅ |

**Totals:** 5 modules · 62 lecture hours · 15 exercises · 5 quizzes.

---

## 🛠️ Projects

Multi-module capstones that integrate the track. Reference implementations live in the
[paired solutions repo](https://github.com/ai-engineering-curriculum/senior-agentic-ai-engineer-solutions).

| Project | Focus | Hours |
|---------|-------|-------|
| [project-401](./projects/project-401-production-subsystem/README.md) | Capstone: lead-build a production multi-agent subsystem | 25h |
| [project-402](./projects/project-402-reliability-hardening/README.md) | Reliability & incident-response hardening | 15h |

- **project-401 — Production Multi-Agent Subsystem.** Implement a production subsystem from a given
  architecture, with a reusable eval harness, OTel tracing, SLOs, cost controls, and durable
  execution. Deliver the implementation, a runbook, and a code-review/standards doc for the team.
- **project-402 — Reliability & Incident-Response Hardening.** Take an existing agent system, define
  SLOs and quality-drift alerts, run a simulated incident (runaway loop / bad output / tool misuse),
  mitigate it, and write the postmortem with durable fixes.

---

## 🎓 Prerequisites

This is the **L40 rung** of the Agentic AI track and assumes you can already build agent and
multi-agent patterns by hand. The intended entry point is the
[**Agentic AI Engineer**](https://github.com/ai-engineering-curriculum/agentic-ai-engineer-learning)
rung (L30) — in particular:

- [mod-204: Multi-Agent Systems Implementation](https://github.com/ai-engineering-curriculum/agentic-ai-engineer-learning/tree/main/lessons/mod-204-multi-agent-implementation)
  — orchestrator-worker loops, handoffs, and sub-agent isolation, implemented by hand.
- [mod-205: Evaluation & Observability](https://github.com/ai-engineering-curriculum/agentic-ai-engineer-learning/tree/main/lessons/mod-205-evaluation-observability)
  — single-agent trajectory/tool-call grading and OpenTelemetry tracing.

You should also be comfortable with:

- `async` Python (type hints) or TypeScript, and writing tests with `pytest` or an equivalent.
- At least one agent framework (LangGraph, the OpenAI Agents SDK, or the Claude Agent SDK), so
  framework-choice tradeoffs are concrete rather than abstract.
- A CI system (GitHub Actions or equivalent) and an observability backend
  (OpenTelemetry plus a metrics/trace store).

See [PREREQUISITES.md](./PREREQUISITES.md) for the full entry-skills checklist.

---

## 🚀 Getting Started

1. **Confirm prerequisites.** Skim [PREREQUISITES.md](./PREREQUISITES.md). If anything is unfamiliar,
   complete the linked [Agentic AI Engineer](https://github.com/ai-engineering-curriculum/agentic-ai-engineer-learning)
   modules first.
2. **Work the modules in order** — mod-401 → mod-405. Later modules lean on earlier ones (you can't
   define SLOs in mod-404 without the traces you built in mod-402).
3. **For each module:** read the lecture chapters, do the exercises, then take the quiz.

   ```bash
   cd lessons/mod-401-agent-systems-in-practice
   cat README.md          # objectives, chapters, exercises, prerequisites
   ```

4. **Attempt exercises before opening solutions.** Reference solutions live in the paired
   [solutions repo](https://github.com/ai-engineering-curriculum/senior-agentic-ai-engineer-solutions) —
   use it to check your work, not to skip it.
5. **Finish with the projects.** Tackle the capstones in `projects/` once the modules are behind you.

---

## 📖 Curriculum Overview

### mod-401 · Implementing Agent Systems from an Architecture — 14h

A reference architecture is a contract, not a blueprint: it fixes boundaries and guarantees but
leaves a hundred implementation decisions to you. This module is about making those decisions
deliberately and owning a subsystem to the point where it is testable, debuggable, and boring to
operate. You turn a reference architecture into a production implementation, make and document
implementation-level tradeoffs (framework, state, tool boundaries) as durable decision records,
refactor a prototype into a maintainable codebase without changing its behavior, and drive
build-vs-buy and complexity decisions at the subsystem level.

[View mod-401 →](./lessons/mod-401-agent-systems-in-practice/README.md)

### mod-402 · Building Eval & Observability Infrastructure — 14h

Where you stop instrumenting one agent and start building the infrastructure every agent team in
your org runs on. You build a reusable evaluation harness (trajectory, tool-call, and LLM-judge
graders behind one stable interface), stand up OpenTelemetry-based tracing and dashboards for agent
*fleets* using the GenAI semantic conventions, define quality signals and regression gates that turn
eval output into a deploy decision, and make eval and observability a paved road other engineers
adopt because it's easier than not.

[View mod-402 →](./lessons/mod-402-eval-observability-infra/README.md)

### mod-403 · Multi-Agent Systems at Scale — 12h

What happens when a multi-agent system meets production traffic, a finite token budget, an SLA, and
inevitable partial failure. You engineer orchestration topologies that degrade gracefully under load
(bounded concurrency, backpressure, timeouts), put real numbers on token economics and latency
budgets, make long-running agent workloads durable and resumable, and harden the seams — inter-agent
messages and tool calls — where production systems actually break. Numbers over vibes: every chapter
gives you a metric to own.

[View mod-403 →](./lessons/mod-403-multi-agent-at-scale/README.md)

### mod-404 · Reliability, Cost & Incident Response — 12h

SRE-for-agents. Agentic systems fail in ways traditional services don't — an agent can return HTTP
200 in 800ms having hallucinated a refund policy or looped 40 times burning $14. You define
**quality** SLOs (not just availability) and alert on drift, wire budgets and circuit breakers that
fail closed into running systems, run incident response for agent-specific failures, and write
blameless postmortems that produce tracked action items that actually land.

[View mod-404 →](./lessons/mod-404-reliability-cost-incident/README.md)

### mod-405 · Technical Leadership for Agentic Teams — 10h

Where you stop being the person who writes the cleverest agent loop and become the person who makes a
*team* ship agentic systems that are safe, reliable, and on-schedule. You lead code review for the
safety failure modes generic review misses (tool authority, prompt-injection surface, loop bounds,
non-determinism), mentor through paved roads and golden-path templates, translate an architect's
design into buildable sequenced work, and scope an agentic initiative so it ships value early and
de-risks the unknowns first.

[View mod-405 →](./lessons/mod-405-technical-leadership/README.md)

---

## 🔗 Paired Solutions Repo

Reference implementations for every exercise and project live in
[`ai-infra-senior-agentic-ai-engineer-solutions`](https://github.com/ai-engineering-curriculum/senior-agentic-ai-engineer-solutions).
Attempt the work here first; use the solutions repo to check your reasoning.

---

## 🗂️ Repository Layout

```text
ai-infra-senior-agentic-ai-engineer-learning/
├── lessons/mod-XXX-*/        modules: lectures, exercises, labs, quizzes
├── projects/project-XXX-*/   multi-module capstones
├── CURRICULUM.md             role-level coverage map
├── PREREQUISITES.md          assumed entry skills
├── VERSIONS.md               release history
└── README.md                 this file
```

---

<!-- aicg:maintained-by -->
Maintained by [VeriSwarm.ai](https://veriswarm.ai)
