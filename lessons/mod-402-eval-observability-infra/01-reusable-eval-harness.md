# Chapter 1 — Building a Reusable Eval Harness

At the junior rung you wrote eval code *inside* an agent's repo: a `test_eval.py` that imports the agent, runs a few cases, and asserts. It works, and it does not travel. The next team writes their own from scratch, with a different dataset format, a different judge prompt, and a different notion of "passing." A **reusable harness** is the asset that ends that duplication: one library that takes any agent plus a dataset and returns a structured, comparable result — so every team grades the same way and you can compare across them.

```text
   dataset ─┐
            ├─▶  Harness.run(agent_fn, dataset)  ─▶  EvalReport
 graders  ──┘         │                                 (scores, per-case
                      ├─ run agent on each case          results, run_id)
                      └─ apply graders to trajectory
```

## The grader interface is the whole design

The harness is only reusable if its graders share one shape. Pick a single protocol and make every grader — trajectory, tool-call, LLM-judge — conform to it. Everything else (datasets, runners, reports) plugs into that contract.

```python
from dataclasses import dataclass
from typing import Protocol

@dataclass
class GradeResult:
    name: str
    score: float          # normalized 0.0–1.0
    passed: bool
    detail: str = ""      # why, for debugging

class Grader(Protocol):
    name: str
    def grade(self, case: "EvalCase", trajectory: "Trajectory") -> GradeResult: ...
```

A `Grader` sees the case (input + expectations) and the captured trajectory (what the agent did), and returns a normalized score. Normalizing every grader to `0.0–1.0` is what lets you average heterogeneous graders into one number and set thresholds uniformly.

## Three grader families behind one interface

- **Trajectory graders** check the *path*: tool-call correctness, in-order subset of required calls, step-budget, no forbidden tools. Reference-based when you have an expected trajectory; reference-free (loop detection, malformed-argument checks) when you don't.
- **Tool-call graders** are a focused trajectory grader: precision/recall over the set of tools the agent invoked versus the set it should have, scored across the dataset.
- **LLM-judge graders** score open-ended quality (helpfulness, faithfulness, no hallucinated citations) by prompting a judge model against a rubric. Wrap them so a flaky judge call degrades to `passed=False` with a detail, never an unhandled exception that sinks the whole run.

```python
class StepBudgetGrader:
    name = "step_budget"
    def __init__(self, max_steps: int):
        self.max_steps = max_steps
    def grade(self, case, trajectory) -> GradeResult:
        n = len(trajectory.steps)
        ok = n <= self.max_steps
        return GradeResult(self.name, 1.0 if ok else 0.0, ok,
                           f"{n} steps (budget {self.max_steps})")
```

## Datasets are versioned artifacts, not literals

A shared harness needs shared, versioned datasets — not cases hard-coded in a test. Store each suite as a file (JSONL is the pragmatic default: one case per line, diffable, streamable) with a stable schema and a version tag. When the dataset changes, the version changes, and every report records which version it ran against. Otherwise "the score dropped" is ambiguous: did the agent regress, or did someone edit the test set?

```python
@dataclass
class EvalCase:
    id: str
    task: str
    expected_tools: list[str] | None = None
    reference: str | None = None
    metadata: dict | None = None
```

## The report is the product

The harness's output is a structured `EvalReport`, not printed lines. It carries a `run_id`, the dataset version, per-case `GradeResult`s, and aggregate scores per grader. Make it serializable (JSON) so it can be stored, compared run-to-run, and — in [Chapter 3](03-quality-signals-and-gates.md) — fed to a gate. Print a human summary on top of it; never *only* print.

## Design rules that make it adoptable

- **Agent-agnostic boundary.** The harness takes a callable `agent_fn(task) -> Trajectory`, not your agent class. Any team that can produce a trajectory can use it.
- **Graders are configured, not coded.** Teams compose graders from the library and set thresholds; they shouldn't fork the harness to add a step budget.
- **Deterministic where you can be.** Pin judge models and temperatures, seed sampling, and record them in the report — eval you can't reproduce can't gate a deploy.
- **Fail soft, report hard.** One case erroring records a failed case and continues; the run still produces a report. A harness that crashes on case 3 of 200 is useless in CI.

## Key takeaways

- A reusable harness is defined by one **grader interface** every grader conforms to and one **agent-agnostic boundary** (`agent_fn -> Trajectory`) every team plugs into.
- Ship **three grader families** — trajectory, tool-call, LLM-judge — all normalized to `0.0–1.0` so scores aggregate and threshold uniformly.
- Treat **datasets as versioned artifacts** and the **`EvalReport` as the product**: structured, serializable, comparable across runs.
- Make graders **configured not coded**, keep eval **reproducible**, and **fail soft** so one bad case never sinks the run.
