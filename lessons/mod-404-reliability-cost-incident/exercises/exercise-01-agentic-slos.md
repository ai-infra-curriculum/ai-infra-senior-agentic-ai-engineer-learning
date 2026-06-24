# exercise-01: Define Agentic SLOs

**Estimated effort:** 3 hours

## Objective

Define a quality-aware SLO for an agent and implement the SLI that backs it. You'll pick indicators that catch quality drift (not just uptime), compute them from a trace dataset, set targets and an error budget, and wire a burn-rate plus baseline-drift alert. By the end you'll be able to answer "is the agent still doing its job well?" with a number, and gate releases on it.

## Background

This exercise covers material from:

- [Chapter 1 — SLOs and SLIs for Agentic Systems](../01-slos-and-slis-for-agents.md)

Use the traces you instrumented in [mod-402](../../mod-402-eval-observability-infra/README.md), or a sample dataset of agent runs with per-run fields for steps, tokens, claims made/grounded, and a sampled eval label.

## Prerequisites

- A dataset (or live stream) of agent run records: at minimum `steps`, `total_tokens`, `grounded_claims`, `total_claims`, and a `judged_success` label on a sampled subset.
- Comfort computing windowed aggregates (any language; examples are Python).
- The Google SRE definitions of SLI / SLO / error budget (see [resources.md](../resources.md)).

## Tasks

### 1. Choose indicators

- Select **three to five** SLIs that proxy quality for your agent from the four families in Chapter 1 (task success, grounding/faithfulness, tool-call validity, loop/efficiency health).
- For each, write one sentence: what counts as a *good* event and what counts as a *valid* event (the denominator). Justify why a pure availability SLI would miss the failure this SLI catches.

### 2. Compute the SLIs from traces

- Implement each SLI as a `good / valid` ratio over a time window.
- For the success-rate SLI, compute it over the **judged subset only** and report the sampling confidence interval; do not dilute with unlabeled runs.
- Handle the empty / no-valid-events case explicitly (do not let a quiet window read as 100% or page on `NaN`).

### 3. Set SLOs and an error budget

- Pick a target and window for each SLI (e.g., grounding ≥ 0.95 over 28 days). Justify the target against your measured baseline — a target below your current performance is meaningless.
- Compute the error budget and state the **release policy**: what happens when the budget is exhausted.

### 4. Alert on drift

- Implement a **burn-rate** alert (fast burn pages, slow burn tickets) against one SLI's error budget.
- Implement a **baseline-drift** alert that compares the current window to a trailing baseline and fires on a significant drop even while still above the absolute SLO.
- Show a case each alert catches that the other misses.

## Starter guidance

```python
from dataclasses import dataclass

@dataclass
class RunRecord:
    grounded_claims: int
    total_claims: int
    steps: int
    total_tokens: int
    judged_success: bool | None  # None where unsampled

def grounding_sli(runs: list[RunRecord]) -> float:
    good = sum(r.grounded_claims for r in runs)
    total = sum(r.total_claims for r in runs)
    return good / total if total else 1.0  # vacuously grounded; document this choice

def loop_health_sli(runs: list[RunRecord], max_steps: int = 12) -> float:
    if not runs:
        return float("nan")  # alerting layer must handle this, not paper over it
    good = sum(1 for r in runs if r.steps <= max_steps)
    return good / len(runs)

def burn_rate(budget_consumed: float, window_fraction: float) -> float:
    """budget_consumed and window_fraction both in [0,1]; >1 means burning too fast."""
    return budget_consumed / window_fraction if window_fraction else float("inf")
```

You do **not** need the cost controls (exercise-02) or a full incident (exercise-03) here — just the measurement and alerting layer.

## Acceptance criteria

You can demonstrate that:

- You have 3–5 quality SLIs, each defined as an explicit `good / valid` ratio, with a written justification for why availability would miss it.
- The success-rate SLI is computed over the judged subset with a reported confidence interval, and every SLI handles the empty case without a false 100% or a spurious page.
- Each SLI has a target, window, and computed error budget, plus a written release policy for budget exhaustion.
- A burn-rate alert and a baseline-drift alert are implemented, and you can show one drift case each one catches that the other does not.

## Reflection

In `NOTES.md`:

1. Which SLI is your strongest leading indicator for a future incident, and why?
2. How did you size the eval sample so the success-rate confidence interval is tighter than your alert threshold? What does it cost per day?
3. Where would a static threshold have fired too late or not at all on your data?

## Stretch goals

- Add a **cost-per-run SLI** (a bridge to exercise-02) and gate releases on it too.
- Backtest your drift alert against a known past regression (e.g., a prompt change) and measure how much sooner it would have fired than a static threshold.
- Split one SLI by tenant or feature and find a segment whose quality is drifting while the global SLI stays green.
