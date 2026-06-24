# exercise-02: Tracing and Dashboards for a Fleet

**Estimated effort:** 5 hours

## Objective

Build the tracing standard a fleet runs on, then prove it answers fleet-level questions. You'll write a thin instrumentation wrapper that emits the OpenTelemetry GenAI semantic conventions, attach resource attributes so traces are sliceable by service, route two simulated services through one collector path into an OTLP-native backend, and build dashboards that answer cost, latency, and error-rate questions *across* services — not for a single run.

## Background

This exercise covers material from:

- [Chapter 2 — Tracing and Dashboards for Agent Fleets](../02-tracing-and-dashboards.md)
- [Chapter 3 — Quality Signals and Regression Gates](../03-quality-signals-and-gates.md) (cost and latency signals come from this trace data)

You'll build on single-agent OTel tracing from the junior rung (mod-205 in the Agentic AI Engineer track). The new requirement is *fleet shape*: a shared wrapper, consistent resource attributes, and dashboards that group by service. Arize Phoenix runs locally and is OTel-native, which makes it the simplest backend for this exercise; Langfuse or LangSmith work equally well.

## Prerequisites

- `opentelemetry-api`, `opentelemetry-sdk`, and the OTLP HTTP exporter installed.
- A local OTLP-native backend (Arize Phoenix is the easiest to run locally) or an account on Langfuse / LangSmith.
- Two small agents (or stubs) that make model calls and tool calls — they represent two fleet services.

## Tasks

### 1. Write the shared instrumentation wrapper

- Write `traced_model_call` and `traced_tool_call` helpers that open spans named `chat {model}` and `execute_tool {tool_name}`.
- Set the GenAI attributes: `gen_ai.operation.name`, `gen_ai.provider.name`, `gen_ai.request.model`, `gen_ai.usage.input_tokens`, `gen_ai.usage.output_tokens`, `gen_ai.tool.name`.
- This wrapper is the asset teams import so they *can't* drift from the keys.

### 2. Set resource attributes per service

- Configure a `TracerProvider` with a `Resource` carrying `service.name`, `service.version`, and `deployment.environment`.
- Give your two services **different** `service.name` values so you can later group by them.

### 3. Route through one export path

- Configure an OTLP exporter (`BatchSpanProcessor`) pointing at your backend (directly, or via a local OTel Collector if you stand one up).
- Run both services; confirm nested traces (root run → model/tool spans) appear in the backend.

### 4. Record errors and sample

- On a failing model/tool call, set the span status to error and attach the exception so the run shows red.
- Add a sampling rule: keep a fraction of normal runs but *always* keep errored runs.

### 5. Build fleet dashboards

- Build (in the backend, or by querying exported spans) charts that group **by `service.name`**: tokens/cost per service, p95 root-span latency per service, and error rate per service.
- Generate enough traffic across both services that the charts are non-trivial, including at least one errored run per service.

## Starter guidance

```python
from opentelemetry import trace
from opentelemetry.sdk.resources import Resource
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter

def init_tracing(service_name: str, version: str, env: str = "production"):
    resource = Resource.create({
        "service.name": service_name,
        "service.version": version,
        "deployment.environment": env,
    })
    provider = TracerProvider(resource=resource)
    provider.add_span_processor(BatchSpanProcessor(
        OTLPSpanExporter(endpoint="http://localhost:4318/v1/traces")))
    trace.set_tracer_provider(provider)
    return trace.get_tracer(service_name)

def traced_model_call(tracer, model, messages, client):
    with tracer.start_as_current_span(f"chat {model}") as span:
        span.set_attribute("gen_ai.operation.name", "chat")
        span.set_attribute("gen_ai.request.model", model)
        # ... call client, set gen_ai.usage.* from the response ...
        raise NotImplementedError
```

You do **not** need the eval harness (exercise-01) here, though the cost/latency you trace becomes signals in exercise-03.

## Acceptance criteria

You can demonstrate that:

- Spans use the GenAI conventions: `chat {model}` / `execute_tool {name}` names and `gen_ai.*` attributes.
- Each service sets `service.name`, `service.version`, and `deployment.environment` as resource attributes.
- Traces from **two** services land in one backend through one export path, with correct parent-child nesting.
- Failing calls show **error status** with the exception attached, and the sampling rule always keeps errored runs.
- You have **fleet** dashboards grouped by `service.name` showing cost, p95 latency, and error rate — not single-run views.

## Reflection

In `NOTES.md`:

1. Which question could you answer *only because* both services used the same `gen_ai.*` keys and resource attributes? What would have broken it?
2. You centralized the export path. What did that let you change once instead of per service?
3. Where would you redact prompt/response content, and why there rather than in each service?

## Stretch goals

- Stand up an actual OTel Collector and move sampling + redaction into the collector config so services inherit it.
- Add a `service.version`-over-time panel and correlate a latency change with a version bump.
- Add tail-based sampling that keeps slow traces (above a p95 threshold) in addition to errored ones.
