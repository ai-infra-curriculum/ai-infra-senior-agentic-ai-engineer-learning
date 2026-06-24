# Resources for mod-403-multi-agent-at-scale (Multi-Agent Systems at Scale)

Primary references for running multi-agent systems at scale. Verify against current docs — agent tooling, model pricing, and provider limits move fast.

## Multi-agent systems and their economics

- **Anthropic — How we built our multi-agent research system** ([anthropic.com/engineering/multi-agent-research-system](https://www.anthropic.com/engineering/multi-agent-research-system)) — a production orchestrator-worker system, including the ~15× token cost of multi-agent vs. chat, the context economics, and the failure modes that justify (or don't) fanning out.
- **Anthropic — Building effective agents** ([anthropic.com/engineering/building-effective-agents](https://www.anthropic.com/engineering/building-effective-agents)) — the pattern taxonomy (orchestrator-workers, routing, evaluator-optimizer) this module hardens for scale.
- **Anthropic — Prompt caching** ([docs.anthropic.com/en/docs/build-with-claude/prompt-caching](https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching)) — the mechanics of caching repeated context, a primary cost lever in Chapter 2.

## Load, backpressure, and reliability

- **Google SRE Book — Handling Overload** ([sre.google/sre-book/handling-overload/](https://sre.google/sre-book/handling-overload/)) — load shedding, graceful degradation, and why a system at capacity should reject fast.
- **Google SRE Book — Addressing Cascading Failures** ([sre.google/sre-book/addressing-cascading-failures/](https://sre.google/sre-book/addressing-cascading-failures/)) — how partial failures cascade and the patterns (timeouts, limits) that contain them.
- **Marc Brooker — Timeouts, retries, and backoff with jitter** ([aws.amazon.com/builders-library/timeouts-retries-and-backoff-with-jitter/](https://aws.amazon.com/builders-library/timeouts-retries-and-backoff-with-jitter/)) — the canonical write-up on retry storms and why jitter matters.
- **Martin Fowler — CircuitBreaker** ([martinfowler.com/bliki/CircuitBreaker.html](https://martinfowler.com/bliki/CircuitBreaker.html)) — the closed/open/half-open state machine from Chapter 4.

## Durable execution

- **Temporal — Documentation** ([docs.temporal.io](https://docs.temporal.io)) — durable execution via deterministic replay; start with workflows vs. activities and the determinism constraints.
- **Temporal — What is durable execution?** ([temporal.io/blog/building-reliable-distributed-systems-in-node](https://temporal.io/blog/building-reliable-distributed-systems-in-node)) — the model behind checkpointing and resumption, and where non-deterministic calls (like the LLM) must live.
- **AWS — Idempotency in the Builders' Library** ([aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/](https://aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/)) — idempotency keys and making retries safe, the rule that makes resumption non-duplicating.

## Observability

- **OpenTelemetry — Documentation** ([opentelemetry.io/docs/](https://opentelemetry.io/docs/)) — spans, attributes, and metrics for attributing tokens, cost, and latency per agent role (used in the budgeting exercise).
- **OpenTelemetry — Semantic conventions for generative AI** ([opentelemetry.io/docs/specs/semconv/gen-ai/](https://opentelemetry.io/docs/specs/semconv/gen-ai/)) — emerging conventions for instrumenting LLM/agent calls (token usage, model, operation) consistently.

> You hardened these patterns by hand in this module — bounded concurrency, durable checkpoints, retries, circuit breakers. When you adopt a framework or platform (Temporal, a managed agent runtime), you'll recognize exactly which of these it's packaging, and you'll debug it faster for having built it yourself.
