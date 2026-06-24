# Module 403: Multi-Agent Systems at Scale — Quiz

10 questions. 70% pass.

### 1. The total in-flight work in an orchestrator-worker system under load is roughly:

- [ ] a) The request rate, independent of fan-out
- [x] b) The product of request rate and per-request fan-out
- [ ] c) The number of worker roles defined
- [ ] d) Fixed by the model provider regardless of traffic

### 2. A process-wide semaphore on worker calls primarily addresses:

- [x] a) Load collapse — it bounds simultaneous calls so queues don't grow without limit
- [ ] b) Malformed messages between agents
- [ ] c) Non-idempotent side effects on retry
- [ ] d) Prompt injection from tool output

### 3. Under overload, the better behavior is to:

- [ ] a) Accept every request and let latency rise until each one eventually finishes
- [x] b) Reject excess requests fast (`429` + `Retry-After`) so accepted work can finish
- [ ] c) Silently drop the slowest workers without telling the caller
- [ ] d) Remove all timeouts so nothing is cut off early

### 4. A worker call with no timeout under load is dangerous because:

- [ ] a) It always returns the wrong answer
- [x] b) A hung worker pins a concurrency slot indefinitely, starving other work
- [ ] c) It doubles the token cost of the run
- [ ] d) It makes the orchestrator non-deterministic

### 5. Compared to a single agent, a multi-agent run typically costs:

- [ ] a) About the same number of tokens
- [ ] b) Fewer tokens, because work is split up
- [x] c) Several times more tokens (Anthropic reported ~15× for their research system)
- [ ] d) Tokens are irrelevant; only latency changes

### 6. Which lever reduces **latency** but not **cost**?

- [x] a) Running independent workers in parallel instead of serially
- [ ] b) Routing easy steps to a cheaper model
- [ ] c) Caching a repeated system prompt
- [ ] d) Bounding the number of workers

### 7. In durable execution, you should checkpoint after each step to durable storage so that:

- [ ] a) The model produces deterministic output
- [x] b) A crash, deploy, or rate-limit resumes from the last completed step instead of restarting from zero
- [ ] c) Tokens are counted more accurately
- [ ] d) Workers can run with a larger context window

### 8. Idempotency matters in resumption because:

- [ ] a) It makes the LLM call deterministic
- [ ] b) It reduces token cost on the first run
- [x] c) A step may run twice (crash after the side effect, before the checkpoint), so the effect must be safe to repeat
- [ ] d) It is only relevant to read-only steps

### 9. The non-deterministic LLM call in a Temporal-style deterministic-replay workflow must:

- [ ] a) Run inline in the workflow function like any other call
- [x] b) Live inside a recorded activity, so replay returns the recorded result instead of re-calling
- [ ] c) Be removed from the workflow entirely
- [ ] d) Be retried until it returns the same output twice

### 10. A circuit breaker on a downstream tool helps where plain retries hurt — namely when:

- [ ] a) The tool returns a single transient `503`
- [x] b) The tool is sustained-down, so retrying just burns the timeout budget and drags every request down
- [ ] c) The tool's output is malformed but the call succeeds
- [ ] d) The tool is faster than expected

---

## Answer key + rationale

1. **b** — Each request fans out to N workers, so concurrent work scales with rate × fan-out; that product is what saturates quota and the event loop.
2. **a** — A semaphore caps simultaneous worker calls, turning "spawn everything" into a controlled queue; it targets load collapse, not the other failure modes.
3. **b** — A fast rejection is recoverable and frees capacity to finish accepted work; a slow timeout that still fails is wasted spend. Never silently drop failures.
4. **b** — Without a timeout a hung worker holds its semaphore slot forever, starving every other request; a hung worker is a failed worker.
5. **c** — Fan-out multiplies tokens; Anthropic reported ~15× for their multi-agent research system, justified only by high task value.
6. **a** — Parallelism shrinks wall-clock (latency) for independent work but spends the same tokens; routing and caching cut cost, and bounding fan-out cuts both.
7. **b** — Checkpointing persists progress so a fresh process resumes from the last completed step rather than re-paying for everything already done.
8. **c** — A process can die after doing the work but before recording it, so resumption re-runs the step; only idempotent steps are safe to repeat.
9. **b** — Deterministic replay re-executes workflow code against recorded history; non-determinism (including the stochastic LLM call) must be a recorded activity or replay diverges.
10. **b** — Retries clear a blip but make a sustained outage worse by burning the timeout budget; a breaker fails fast and lets the dependency recover.
