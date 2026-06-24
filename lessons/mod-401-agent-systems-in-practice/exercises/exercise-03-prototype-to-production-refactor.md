# exercise-03: Prototype to Production Refactor

**Estimated effort:** 3 hours

## Objective

Take a single-file prototype agent — prompt, model call, tool loop, parsing, and a hard-coded key all fused together — and refactor it into a layered, tested codebase **without changing its observable behavior.** You will pin the behavior with characterization tests against a faked model, pull the fused concerns into separate layers, and reach the agent loop you could not test before. The skill being graded is safe restructuring under a behavior-preservation constraint, the everyday reality of inheriting a teammate's spike.

## Background

This exercise covers material from:

- [Chapter 3 — Refactoring a Prototype into a Maintainable Codebase](../03-prototype-to-production.md)
- [Chapter 4 — Build-vs-Buy and Complexity Decisions](../04-build-vs-buy-and-complexity.md)

A prototype proved an idea; it did not produce something a team can own. Refactoring it is a distinct discipline from writing it: the rule is that the agent's outputs do not change, so you can always tell whether you broke something. Improving the outputs is a separate task for a separate day.

## Prerequisites

- A single-file prototype agent. Use your own from an earlier module, or write a ~80-line one first: a loop that takes a question, calls a model, optionally calls one or two tools, and returns an answer — with the model client, prompt, tool dispatch, and config all inline.
- A test runner and the ability to substitute a fake for the model client.
- A model whose responses you can record and replay (capture real responses once, then replay), so tests are deterministic.

## Tasks

### 1. Pin behavior with characterization tests

- Build a `FakeModel` that replays recorded responses for fixed inputs, removing nondeterminism.
- Capture the prototype's current behavior on 3–5 representative inputs as characterization tests. Assert on structure — the sequence of tool calls and the final shape — plus exact text only where the model is pinned by the fake.
- These tests must stay green through every later step. Green means behavior preserved.

### 2. Extract configuration

- Move keys, model name, and limits out of the source into a typed config object loaded from the environment. Removing the hard-coded key is both the first refactor step and a security fix.
- Run the characterization tests. Still green.

### 3. Introduce I/O interfaces

- Define a `ModelClient` interface and adapt the real client to it; do the same for tool transport.
- Make the loop depend on the interfaces, not concrete clients. Now `FakeModel` plugs in as the `ModelClient` for tests.
- Still green.

### 4. Lift the loop into its own layer

- Move the reason-act loop into an `agent` module that depends only on interfaces and pure domain functions (parsing, validation, shaping).
- With the loop now isolated, add **new** tests it could not have before: assert tool-call order on a scripted fake, and exercise a partial-failure path (a tool errors mid-run).
- Characterization tests still green; new loop tests also green.

### 5. Justify your stopping point

- Stop when the parts that genuinely need to be swapped or tested are reachable. Write a short note naming one abstraction you deliberately did **not** add, applying YAGNI/KISS, and why adding it would have been speculative complexity.

## Starter guidance

There is no single correct target structure, but a healthy end state looks like this — the rubric below is what graders check, not line counts.

```text
agent/
├── config.py     # typed config from env; no secrets in source
├── clients.py    # ModelClient + ToolTransport interfaces + real adapters
├── agent.py      # the reason-act loop; depends on interfaces only
├── domain.py     # pure parse / validate / shape functions
└── tests/
    ├── fakes.py                  # FakeModel (replay) + tool fakes
    ├── test_characterization.py  # behavior pinned from step 1, stays green
    └── test_loop.py              # new: tool order, partial failure
```

Refactor rubric (self-score each move):

```text
| Property                          | Before | After |
| --------------------------------- | ------ | ----- |
| Secrets out of source             |   no   |  yes  |
| Loop testable without live API    |   no   |  yes  |
| Concerns in separate layers       |   no   |  yes  |
| Behavior unchanged (char. tests)  |   —    | green |
| Partial-failure path covered      |   no   |  yes  |
```

Work one row at a time: make the change, run characterization tests, commit only on green. A red run localizes exactly where behavior shifted.

## Acceptance criteria

You can demonstrate that:

- Characterization tests captured before refactoring stay green through the final commit — behavior is provably unchanged.
- No secret is hard-coded; config loads from the environment.
- The agent loop depends on a `ModelClient` interface and runs under `FakeModel` with no network.
- New tests assert tool-call order and a partial-failure path that were untestable in the prototype.
- Your stopping-point note names a concrete abstraction you declined to add and justifies it with YAGNI/KISS.

## Reflection

In `NOTES.md`:

1. Which step turned the characterization tests red first, and what did that tell you about a hidden coupling in the prototype?
2. Before the refactor, what specifically prevented you from testing the loop's partial-failure path?
3. Name the abstraction you were most tempted to add for "flexibility" and chose not to. What concrete future event would justify adding it later?

## Stretch goals

- Add a second `ModelClient` adapter for a different provider and prove the loop and all tests are unchanged — the payoff of the interface seam.
- Introduce a property-based test over the pure `domain.py` functions (parsing/validation) to find edge cases your example-based tests missed.
- Measure and report the test suite's wall-clock before and after `FakeModel` — quantify how much faster the suite got once the live API left the loop.
