# Resources for mod-404-reliability-cost-incident

Primary references for reliability, cost, and incident response on agentic systems. Verify against current docs — agent tooling and pricing move fast.

## SLOs, SLIs, and error budgets

- **Google — Site Reliability Engineering, "Service Level Objectives"** ([sre.google/sre-book/service-level-objectives](https://sre.google/sre-book/service-level-objectives/)) — the canonical definitions of SLI, SLO, and error budget. Read this first; Chapter 1 maps it onto agents.
- **Google — The Site Reliability Workbook, "Alerting on SLOs"** ([sre.google/workbook/alerting-on-slos](https://sre.google/workbook/alerting-on-slos/)) — burn-rate and multi-window alerting, the basis for the drift alerts in Chapter 1 and exercise-01.
- **Google — The Site Reliability Workbook, "Implementing SLOs"** ([sre.google/workbook/implementing-slos](https://sre.google/workbook/implementing-slos/)) — practical SLI selection and target setting.

## Observability and instrumentation

- **OpenTelemetry** ([opentelemetry.io/docs](https://opentelemetry.io/docs/)) — the vendor-neutral standard for traces and metrics. Your SLIs and cost records ride on this.
- **OpenTelemetry — Semantic conventions for GenAI / LLM spans** ([opentelemetry.io/docs/specs/semconv/gen-ai](https://opentelemetry.io/docs/specs/semconv/gen-ai/)) — standard attribute names (model, token counts, tool calls) so your cost and quality telemetry is portable.

## Incident response and on-call

- **Google — SRE book, "Managing Incidents"** ([sre.google/sre-book/managing-incidents](https://sre.google/sre-book/managing-incidents/)) — incident command structure (commander, ops, comms) and declaring early; the loop Chapter 3 builds on.
- **Google — SRE book, "Being On-Call"** ([sre.google/sre-book/being-on-call](https://sre.google/sre-book/being-on-call/)) — sustainable on-call, the human side of the rotation a senior owns.
- **PagerDuty — Incident Response documentation** ([response.pagerduty.com](https://response.pagerduty.com/)) — a practical, open playbook for roles, severities, and during-incident process.

## Postmortems

- **Google — SRE book, "Postmortem Culture: Learning from Failure"** ([sre.google/sre-book/postmortem-culture](https://sre.google/sre-book/postmortem-culture/)) — the blameless principle and tracked action items; the spine of Chapter 4.
- **Google — Example postmortem** ([sre.google/sre-book/example-postmortem](https://sre.google/sre-book/example-postmortem/)) — a worked template to adapt for the agent-specific fields in Chapter 4.

## Agent-specific failure modes and cost

- **Anthropic — Building effective agents** ([anthropic.com/engineering/building-effective-agents](https://www.anthropic.com/engineering/building-effective-agents)) — agent patterns and the loop/stopping-condition concerns behind runaway-loop and budget design.
- **Anthropic — How we built our multi-agent research system** ([anthropic.com/engineering/multi-agent-research-system](https://www.anthropic.com/engineering/multi-agent-research-system)) — real token-economics and failure modes of a production multi-agent system; context for cost controls.
- **OWASP — Top 10 for LLM Applications** ([owasp.org/www-project-top-10-for-large-language-model-applications](https://owasp.org/www-project-top-10-for-large-language-model-applications/)) — including Unbounded Consumption and Excessive Agency, the security framing for the cost guards (Chapter 2) and tool-misuse incidents (Chapter 3).
