# Implementing Agent Systems from an Architecture quiz

A knowledge check for mod-401. Questions are judgment-oriented — most have a defensible best answer rather than a single mechanical one. Answers and rationale are at the bottom; decide before you scroll.

## Questions

### 1. Reading the architecture

An architect's diagram says your executor box "must be idempotent on the call id." Where does this requirement most belong in your implementation?

- A. A sentence in the system prompt telling the model not to repeat calls.
- B. A defensive `if` buried in the transport.
- C. The durable state layer, keyed by call id, with a test asserting replays are no-ops.
- D. A comment, since the planner already prevents duplicate calls.

### 2. The interface stub

Why is shipping a contract-honoring `StubExecutor` on day one valuable, beyond "it compiles"?

- A. It is the executable form of the contract, so neighbor teams build against it in parallel and your real internals stay swappable for it.
- B. It permanently replaces the need for a real implementation.
- C. It makes the model's outputs more accurate.
- D. It removes the need for an interface.

### 3. Framework choice

You pick LangGraph for a subsystem. What single practice keeps that choice reversible?

- A. Use every LangGraph primitive directly throughout the subsystem.
- B. Wrap LangGraph behind your own interface so it lives behind one adapter and never leaks into the contract.
- C. Avoid writing any tests until the framework is proven.
- D. Copy the framework's types into your public API.

### 4. State management

A long-running planner must survive deploys. Which split is correct?

- A. Put everything, including the per-turn scratchpad, in Postgres for safety.
- B. Keep everything in memory; restarts are rare.
- C. Keep the per-turn scratchpad in-process; put the agreed, resumable task state in durable storage as the single source of truth.
- D. Store task state in the system prompt.

### 5. Tool boundaries

You must expose a powerful capability. Where does the authorization check belong?

- A. In the system prompt, instructing the model to only act within scope.
- B. In the tool's executor, as an enforced check — a prompt is a suggestion, an executor check is a guarantee.
- C. Nowhere; the model is trusted.
- D. In the planner only.

### 6. Refactoring safely

You are refactoring a prototype agent. Mid-refactor you notice its output is subtly wrong. What do you do?

- A. Fix the output in the same refactor — it is clearly a bug.
- B. Keep behavior unchanged (characterization tests stay green) and file the correctness fix as separate work.
- C. Delete the characterization tests so they stop failing.
- D. Rewrite the whole subsystem from scratch.

### 7. Characterization tests

What makes characterization tests possible for an otherwise nondeterministic agent?

- A. Running the live model many times and averaging.
- B. A faked, replayable model plus asserting on structure (tool calls, shape) where the model is genuinely free.
- C. Asserting exact prose for every output.
- D. Disabling all tools.

### 8. Build-vs-buy

You need retry-with-backoff. A mature, maintained library does it well and your needs are standard. The senior default?

- A. Hand-roll it to avoid a dependency.
- B. Adopt the library behind your own interface ("wrap, don't marry").
- C. Adopt the library and depend on its types directly everywhere.
- D. Skip retries entirely.

### 9. Complexity

What is the test for whether an abstraction you are about to add earns its keep?

- A. Whether it might be useful someday.
- B. Whether you can name the concrete, present pressure it relieves.
- C. Whether it makes the code look more sophisticated.
- D. Whether the framework documentation uses it.

### 10. Ownership

You "own a subsystem end to end." Which artifact set best reflects that ownership in production?

- A. Working code and nothing else.
- B. Code plus a stable interface, a failure policy, boundary observability, and a runbook an operator can follow without you.
- C. A large suite of speculative abstractions for future flexibility.
- D. A long design doc with no tests.

## Answer key

1. **C** — Idempotency is a storage decision, not a prompt instruction; prove it with a replay-is-a-no-op test. (Chapter 2)
2. **A** — The stub is the executable contract; it unblocks neighbors and your real executor must stay substitutable for it. (Chapter 1)
3. **B** — Wrapping the framework behind your interface keeps the choice reversible and out of the contract. (Chapter 2)
4. **C** — Split ephemeral scratch (in-process) from durable, resumable task state (single source of truth). (Chapter 2)
5. **B** — Enforce policy in the executor; a prompt is a suggestion, an executor check is a guarantee. (Chapter 2)
6. **B** — A refactor preserves observable behavior; correctness fixes are separate work, or you lose the ability to tell what you broke. (Chapter 3)
7. **B** — A replayable fake plus structural assertions removes nondeterminism without over-pinning free text. (Chapter 3)
8. **B** — Adopt the battle-tested library, but wrap it so the dependency stays contained and reversible. (Chapter 4)
9. **B** — Naming the concrete present pressure separates an earned seam from speculative generality (YAGNI/KISS). (Chapter 4)
10. **B** — Ownership in production is code *plus* the contract, failure policy, observability, and a runbook. (Chapters 1, 2, 4)
