# Chapter 2 — Tracing and Dashboards for Agent Fleets

Tracing one agent is solved: open a span per model call and tool call, export over OTLP, read the tree. Tracing a *fleet* is a different problem. Twelve services from six teams, each emitting spans, is only useful if they emit spans the *same way* — same attribute keys, same resource labels, same notion of what a "run" is. Otherwise no dashboard can answer "which agent is burning the most tokens?" or "is latency up across the fleet or just one service?" The infrastructure you build here is the **shared instrumentation standard** plus the **fleet dashboards** that standard makes possible.

```text
   svc-research ─┐
   svc-support  ─┤──▶  OTel Collector  ──▶  backend (Langfuse / Phoenix / LangSmith)
   svc-ops      ─┘     (one pipeline)         │
                                              └─▶  fleet dashboards
                                                  (cost, latency, error rate by service)
```

## Standardize on the GenAI semantic conventions

The OpenTelemetry **GenAI semantic conventions** define standard span names and `gen_ai.*` attribute keys. Mandating them across the fleet is what makes cross-service dashboards possible — backends compute cost, latency, and token charts automatically when the keys are standard. The ones every service must emit:

- `gen_ai.operation.name` — `chat`, `execute_tool`, `embeddings`.
- `gen_ai.provider.name` (and/or `gen_ai.system`) — `anthropic`, `openai`.
- `gen_ai.request.model` / `gen_ai.response.model` — requested vs. served model.
- `gen_ai.usage.input_tokens` / `gen_ai.usage.output_tokens` — for cost rollups.
- `gen_ai.tool.name` — on `execute_tool` spans.

Span names follow the convention too: `chat {model}` and `execute_tool {tool_name}`. The infra contribution is not instrumenting one service — it is publishing a thin wrapper that every team imports so they *can't* drift from the keys.

## Resource attributes are how you slice a fleet

Span attributes describe one operation; **resource attributes** describe the *thing emitting* the spans, and they are what let you group a fleet. Set them once per service, at startup, and every span inherits them.

```python
from opentelemetry.sdk.resources import Resource
from opentelemetry.sdk.trace import TracerProvider

resource = Resource.create({
    "service.name": "svc-research-agent",
    "service.version": "2.3.1",
    "deployment.environment": "production",
})
provider = TracerProvider(resource=resource)
```

Now every dashboard can group by `service.name`, compare `production` against `staging` via `deployment.environment`, and correlate a regression with a `service.version` bump. Without consistent resource attributes you have a pile of spans you can't partition.

## One collector, swappable backends

Point every service at a shared **OTel Collector** rather than wiring each one directly to a backend. The collector centralizes the OTLP endpoint, applies sampling and redaction policy once, and lets you swap or fan out backends without touching service code.

```python
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.trace.export import BatchSpanProcessor

exporter = OTLPSpanExporter(endpoint="http://otel-collector:4318/v1/traces")
provider.add_span_processor(BatchSpanProcessor(exporter))
```

Because every service used the GenAI conventions, the backend (Langfuse, Arize Phoenix, or LangSmith — all OTLP-native) renders them without per-service configuration. Switching backends is a collector config change, not a fleet-wide re-instrumentation.

## Dashboards answer fleet questions, not run questions

A single trace answers "what did *this* run do." A fleet dashboard answers operational questions, and you should design it around the four that matter:

- **Cost** — tokens and dollars per service per day, derived from `gen_ai.usage.*`. The chart that catches a model-config mistake before the invoice does.
- **Latency** — p50/p95/p99 of root-span duration, grouped by `service.name`. Tail latency is where agents hurt users.
- **Error rate** — fraction of root spans with error status, per service. Spikes here page someone.
- **Throughput** — runs per minute, to read the others in context (a latency spike at 10x traffic is different from one at baseline).

## Sampling and redaction are fleet policy, not per-team choices

At fleet scale, tracing every run at full fidelity is expensive and leaks data. Set policy centrally in the collector: **tail-sample** to keep a representative fraction plus *always* keep errored and high-latency traces, and **redact** prompt/response content that may contain PII before it reaches a backend. Making this a collector policy means a new team inherits safe defaults instead of each team rediscovering the rule.

## Key takeaways

- A fleet is traceable only when every service emits the **GenAI semantic conventions** — ship a wrapper so teams can't drift from the `gen_ai.*` keys.
- **Resource attributes** (`service.name`, `deployment.environment`, `service.version`) are how you slice and group a fleet; set them once at startup.
- Route every service through **one OTel Collector** so sampling, redaction, and backend choice are central policy and backends stay swappable.
- Build **fleet dashboards** around cost, latency, error rate, and throughput — and enforce **sampling and redaction** as collector policy, not per-team habit.
