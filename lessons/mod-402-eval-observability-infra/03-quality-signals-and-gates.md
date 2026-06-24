# Chapter 3 — Quality Signals and Regression Gates

A harness that produces an `EvalReport` ([Chapter 1](01-reusable-eval-harness.md)) and dashboards that show fleet health ([Chapter 2](02-tracing-and-dashboards.md)) are observability — they tell you the truth *after* you ask. A **gate** is control: it turns that truth into a deploy decision and refuses to merge a change that regresses quality. This is the piece that protects a fleet, because the alternative is discovering a regression from users instead of from CI.

```text
   PR ─▶ run harness ─▶ EvalReport ─▶ compare to baseline ─▶ gate
                                                              ├─ pass ─▶ merge
                                                              └─ fail ─▶ block + report
```

## A signal is a scored answer to one question

Don't gate on a wall of raw numbers. Define a small set of **quality signals**, each a single scored answer to a question a reviewer would ask: *Is it correct? Does it take a sane path? Is it cheap enough? Is it fast enough?* Each signal aggregates one or more graders into a `0.0–1.0` score plus a pass/fail against a threshold.

```python
from dataclasses import dataclass

@dataclass
class Signal:
    name: str            # "answer_quality", "tool_correctness", "cost", "latency"
    score: float         # 0.0–1.0, aggregated from graders or run stats
    threshold: float
    @property
    def passed(self) -> bool:
        return self.score >= self.threshold
```

Four signals — answer quality, tool/trajectory correctness, cost, latency — cover most agents. Cost and latency come from the trace data ([Chapter 2](02-tracing-and-dashboards.md)); quality and correctness come from the harness. One framework, both data sources.

## Absolute thresholds vs. regression thresholds

There are two ways to set the bar, and you usually want both:

- **Absolute threshold** — "tool correctness must be ≥ 0.90." Simple, but brittle: too high and good PRs flake; too low and real regressions slip through.
- **Regression threshold** — "this PR's score must not drop more than 2 points below the baseline (the last green run on `main`)." This is what actually protects a fleet, because it adapts as the agent improves and catches *deltas* rather than fixed lines.

```python
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
```

Store the baseline as an artifact updated on every green merge to `main`, so the gate always compares against the last known-good state.

## Don't let the gate flake

A gate that fails randomly gets disabled within a week — and a disabled gate protects nothing. LLM evals are noisy, so engineer the gate to be stable:

- **Run enough cases.** A 10-case suite swings wildly; a few hundred cases makes the aggregate score stable. Signal noise falls with dataset size.
- **Pin the judge.** Fix the judge model and temperature so the same PR scores the same twice. A drifting judge is indistinguishable from a drifting agent.
- **Use a tolerance band.** The `max_drop` above absorbs benign noise; set it from the suite's measured run-to-run variance, not a guess.
- **Quarantine flaky cases.** A case that flips pass/fail across identical runs is a broken case, not a signal — pull it from the gating set and fix it offline.

## Report the failure where the engineer is

When the gate blocks, it must say *what* regressed and point at *evidence* — not just exit non-zero. Post a PR comment with the failing signals, the score deltas, and links to the worst offending traces in your backend. A gate that fails with "eval failed" trains engineers to retry until green; a gate that says "tool_correctness dropped 0.06; here are the 3 cases that newly fail and their traces" gets the bug fixed.

## Gate in CI, not after deploy

The gate belongs in the pull-request pipeline, before merge — the cheapest place to catch a regression. Keep a faster **smoke suite** (tens of cases, no judge) on every push for quick feedback, and run the **full suite** (hundreds of cases, with judge) as the merge-blocking gate. Reserve the slowest, most expensive evals for a nightly run against `main` that watches for slow drift the per-PR gate can't afford to measure.

## Key takeaways

- A **gate** turns eval output into a deploy decision; without it, observability tells you about a regression only after users do.
- Express quality as a few **signals** (answer quality, tool correctness, cost, latency), each a `0.0–1.0` score against a threshold, sourced from the harness and the traces.
- Gate on both **absolute** and **regression** thresholds — the regression delta vs. the last green baseline is what actually protects the fleet.
- Engineer the gate to **not flake** (enough cases, pinned judge, tolerance band, quarantine), **report failures with evidence**, and run it **in CI before merge**.
