# exercise-03: Regression Gates in CI

**Estimated effort:** 4 hours

## Objective

Turn the eval harness from exercise-01 into a deploy gate that blocks a regression in CI. You'll define quality signals with both absolute and regression thresholds, store a baseline as an artifact updated on green merges, wire the gate into a pull-request pipeline, and prove it by submitting a PR that regresses the agent and watching the gate block it with an evidence-rich failure report.

## Background

This exercise covers material from:

- [Chapter 3 — Quality Signals and Regression Gates](../03-quality-signals-and-gates.md)
- [Chapter 1 — Building a Reusable Eval Harness](../01-reusable-eval-harness.md) (the gate consumes its `EvalReport`)

The harness already produces structured reports; this exercise is about *control* — converting a report into a merge decision that is stable enough teams won't disable it. Use GitHub Actions or any CI you can run; the gate logic is portable.

## Prerequisites

- The reusable harness and `EvalReport` from exercise-01 (or an equivalent that emits per-grader aggregate scores).
- A CI system you can configure to run on pull requests (GitHub Actions, GitLab CI, etc.).
- A dataset large enough (aim for a few hundred cases, even if stubbed) that aggregate scores are stable.

## Tasks

### 1. Define signals from the report

- Map harness aggregates and trace stats onto a small set of `Signal`s: `answer_quality`, `tool_correctness`, `cost`, `latency`.
- Each signal has a `0.0–1.0` score and an absolute threshold.

### 2. Implement the gate logic

- Write `gate(current_signals, baseline, max_drop)` that fails if any signal is below its absolute threshold **or** drops more than `max_drop` below the baseline.
- Return a clear pass/fail plus the list of specific failures (which signal, current vs. threshold, and delta vs. baseline).

### 3. Manage the baseline

- Store the baseline (per-signal scores from the last green run on `main`) as a CI artifact or committed file.
- On a green merge to `main`, update the baseline. On a PR, compare against it — do **not** overwrite it from a PR run.

### 4. Wire it into CI

- Add a smoke suite (tens of cases, no judge) that runs on every push for fast feedback.
- Add the full suite (hundreds of cases, with judge) as the **merge-blocking** gate job on pull requests.

### 5. Prove it blocks a regression

- Submit a PR that deliberately regresses the agent (e.g., remove a required tool call so `tool_correctness` drops).
- Show the gate **fails** the PR and posts a comment naming the regressed signal, the delta, and links/ids of the newest-failing cases.
- Submit a benign PR and show the gate **passes** without flaking.

## Starter guidance

```python
from dataclasses import dataclass

@dataclass
class Signal:
    name: str
    score: float
    threshold: float
    @property
    def passed(self) -> bool:
        return self.score >= self.threshold

def gate(current: list[Signal], baseline: dict[str, float],
         max_drop: float = 0.02) -> tuple[bool, list[str]]:
    failures = []
    for s in current:
        if not s.passed:
            failures.append(f"{s.name} {s.score:.3f} < threshold {s.threshold:.3f}")
        drop = baseline.get(s.name, s.score) - s.score
        if drop > max_drop:
            failures.append(f"{s.name} regressed {drop:.3f} vs baseline")
    return (len(failures) == 0, failures)

# In CI: build signals from the EvalReport, load baseline, call gate(),
# exit non-zero on failure, and post the failures as a PR comment.
```

You do **not** need to rebuild the harness (exercise-01) or tracing (exercise-02) — consume their outputs.

## Acceptance criteria

You can demonstrate that:

- Signals are derived from the harness report (and trace stats for cost/latency), each normalized with a threshold.
- The gate fails on either an **absolute** breach or a **regression** beyond `max_drop` vs. the baseline.
- The baseline updates only on green merges to `main`, never from a PR run.
- A smoke suite runs on every push; the full suite is the merge-blocking gate on PRs.
- A regressing PR is **blocked** with a comment naming the signal, the delta, and the newly-failing cases; a benign PR **passes** without flaking.

## Reflection

In `NOTES.md`:

1. How did you choose `max_drop`? Show the run-to-run variance you measured and why your band sits above it.
2. What happens to the gate when the dataset version changes? How do you avoid mistaking a dataset edit for a regression?
3. A teammate wants to bypass the gate "just this once." What do you allow, what do you log, and why?

## Stretch goals

- Add a nightly full-fidelity eval against `main` that catches slow drift the per-PR gate can't afford to measure.
- Add per-case quarantine: detect cases that flip pass/fail across identical runs and exclude them from gating until fixed.
- Make the PR comment link directly to the worst offending traces in your exercise-02 backend.
