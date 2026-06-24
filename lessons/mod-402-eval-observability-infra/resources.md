# Resources for mod-402-eval-observability-infra

Primary references for building eval and observability infrastructure. Verify against current docs — the OTel GenAI conventions and eval tooling are evolving fast.

## OpenTelemetry GenAI conventions

- **OpenTelemetry — Semantic conventions for generative AI** ([opentelemetry.io/docs/specs/semconv/gen-ai](https://opentelemetry.io/docs/specs/semconv/gen-ai/)) — the standard `gen_ai.*` span names and attribute keys. The vocabulary your whole fleet must speak. Start here.
- **OpenTelemetry — GenAI agent and tool spans** ([opentelemetry.io/docs/specs/semconv/gen-ai/gen-ai-agent-spans](https://opentelemetry.io/docs/specs/semconv/gen-ai/gen-ai-agent-spans/)) — conventions specific to agent runs, tool execution, and `execute_tool` spans.
- **OpenTelemetry — Resource semantic conventions** ([opentelemetry.io/docs/specs/semconv/resource](https://opentelemetry.io/docs/specs/semconv/resource/)) — `service.name`, `service.version`, `deployment.environment`: the keys that let you slice a fleet.
- **OpenTelemetry Collector** ([opentelemetry.io/docs/collector](https://opentelemetry.io/docs/collector/)) — the one pipeline where sampling and redaction policy live; processors, exporters, and pipelines.

## Observability platforms (OTLP-native)

- **Arize Phoenix** ([docs.arize.com/phoenix](https://docs.arize.com/phoenix)) — open-source, OTel/OpenInference-native, runs locally. The easiest backend for the exercises.
- **Langfuse** ([langfuse.com/docs](https://langfuse.com/docs)) — open-source LLM engineering platform with a native OTLP endpoint, plus datasets and evals.
- **LangSmith** ([docs.smith.langchain.com](https://docs.smith.langchain.com)) — tracing, datasets, and evaluation; first-class for LangChain/LangGraph and ingests OTel traces.

## Evaluation frameworks

- **RAGAS** ([docs.ragas.io](https://docs.ragas.io)) — reference-free and reference-based metrics (faithfulness, answer relevancy, context precision/recall) you can wrap behind your grader interface.
- **OpenAI Evals** ([github.com/openai/evals](https://github.com/openai/evals)) — a framework and registry for building and running evals; useful as a structural reference for harness design.
- **DeepEval** ([github.com/confident-ai/deepeval](https://github.com/confident-ai/deepeval)) — a `pytest`-style LLM eval framework; a model for making eval feel like familiar testing.
- **Anthropic — Building effective agents** ([anthropic.com/engineering/building-effective-agents](https://www.anthropic.com/engineering/building-effective-agents)) — the evaluator-optimizer pattern and why evaluation belongs in the agent loop.

## Practices

- **Hamel Husain — Your AI product needs evals** ([hamel.dev/blog/posts/evals](https://hamel.dev/blog/posts/evals/)) — why bespoke, dataset-backed evals beat generic metrics, and how to make eval a habit a team keeps.
- **Google SRE Workbook — Service Level Objectives** ([sre.google/workbook/implementing-slos](https://sre.google/workbook/implementing-slos/)) — the discipline behind turning signals into thresholds and gates; read it as the model for your quality signals.

> You're building infrastructure, not a one-off script. Where a platform packages one of these patterns (a hosted harness, a managed collector), adopt it deliberately — but instrument with the OTel conventions so the backend stays swappable and the gate stays yours.
