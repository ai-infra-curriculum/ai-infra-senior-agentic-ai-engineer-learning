# Chapter 3 — Refactoring a Prototype into a Maintainable Codebase

Most agent subsystems start life as a prototype: a single file where the prompt, the model call, the tool loop, the parsing, and a hard-coded API key all live together because that is how you got it working fast. The prototype proved the idea. It is also nearly impossible to test, change, or operate — and turning it into something a team can own is a distinct skill from writing it in the first place.

The governing constraint of this work is the one rule that makes refactoring safe: **observable behavior must not change.** A refactor that "improves" the agent's outputs is not a refactor; it is a rewrite wearing a refactor's clothes, and you have lost the ability to tell whether you broke something. Hold behavior constant, restructure underneath, prove equivalence.

## Pin behavior before you touch anything

You cannot refactor safely what you cannot characterize. Before moving a single line, capture the prototype's current behavior as **characterization tests**: feed it a handful of representative inputs, record exactly what it produces, and freeze those as assertions. They are not testing *correctness* — the prototype may well be subtly wrong — they are testing *sameness*. As long as they stay green, your restructuring has not changed what the system does.

```python
# Characterization test: pins current behavior, right or wrong.
def test_summarize_prototype_unchanged(snapshot):
    out = run_prototype(FIXED_INPUT, seed=0, model=FAKE_MODEL)
    assert out == snapshot  # green == behavior preserved
```

Two things make this tractable for an agent, which is otherwise nondeterministic: **seed/freeze the model** behind a fake that replays recorded responses, and **assert on structure** (the tool calls made, the final shape) rather than exact prose where the model is genuinely free. Without a fake model, you have no characterization tests and therefore no safe refactor.

## Separate the layers the prototype fused

A prototype fuses concerns that production needs apart. The refactor is mostly the act of pulling them into layers, each independently testable:

- **Configuration** — keys, model names, limits. Out of the source, into environment or a typed config object. A hard-coded key is both a refactor blocker and a security incident waiting to happen; this is the first thing to extract.
- **I/O adapters** — the model client and the tool transports. Behind interfaces so a fake can replace them in tests.
- **Orchestration** — the agent loop itself: reason, call tool, observe, repeat. This is the logic you actually want to test, and it cannot be tested while it is welded to a live API.
- **Domain logic** — parsing, validation, result shaping. Pure functions where possible; pure functions are the cheapest thing in the world to test.

```text
   prototype.py  (everything fused)
        │  refactor
        ▼
   config.py        ← keys, models, limits
   clients.py       ← model + tool adapters (interfaces)
   agent.py         ← the loop  (depends on interfaces, not clients)
   domain.py        ← pure parse/validate/shape functions
   tests/           ← fake model + tool fakes wire it all together
```

The payoff is concentrated in `agent.py`. Once the loop depends on a `ModelClient` interface instead of `openai.chat.completions`, you can drive it with a scripted fake, assert which tools it called in which order, and test the partial-failure and retry paths that were untestable before. That is the whole reason to refactor: not aesthetics, but reaching the logic that matters.

## Refactor in small, green steps

Do not rewrite. Move one concern at a time, run the characterization tests after each move, and commit when green. Extract config — green — commit. Introduce the `ModelClient` interface and adapt the real client to it — green — commit. Lift the loop into `agent.py` — green — commit. If a step turns the tests red, you have found exactly where behavior changed, in a diff small enough to understand. This is the discipline that lets you refactor a subsystem other people depend on without a scary big-bang merge.

## Know when to stop

Maintainability is a means, not an end. Three injected dependencies and four small modules is a healthy executor; twelve interfaces and a plugin registry for a subsystem that calls one model and one tool is the over-engineering this curriculum's complexity rules warn against (see [Chapter 4](04-build-vs-buy-and-complexity.md)). Stop when the layers you actually need to swap or test are swappable and testable. Every seam you add is a seam someone must understand. Add the ones the subsystem's real change-and-test pressure demands, and no more.

## Key takeaways

- The rule of refactoring is that observable behavior does not change — pin it with characterization tests against a faked, replayable model before touching code.
- Pull the fused prototype into layers: config, I/O adapters behind interfaces, the orchestration loop, and pure domain functions.
- The payoff is testability of the loop — depend on a `ModelClient` interface so a scripted fake can exercise tool order, retries, and partial failure.
- Move one concern at a time, stay green, commit small; a red step localizes exactly where behavior shifted.
- Stop adding seams when the parts that genuinely need to change and be tested are reachable — extra abstraction is extra cost.
