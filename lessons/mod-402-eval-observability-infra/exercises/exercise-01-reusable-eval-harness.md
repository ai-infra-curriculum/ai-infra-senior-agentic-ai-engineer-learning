# exercise-01: Build a Reusable Eval Harness

**Estimated effort:** 5 hours

## Objective

Build an eval harness that two *different* agents can share without modification. You'll define one grader interface, implement three grader families (trajectory, tool-call, LLM-judge) behind it, load cases from a versioned dataset file, and emit a structured `EvalReport`. The point is not to evaluate one agent well — it's to build the reusable asset a team adopts, where adding a new agent means pointing the harness at it, not rewriting it.

## Background

This exercise covers material from:

- [Chapter 1 — Building a Reusable Eval Harness](../01-reusable-eval-harness.md)
- [Chapter 3 — Quality Signals and Regression Gates](../03-quality-signals-and-gates.md) (the `EvalReport` you produce here is the gate's input)

You'll reuse the trajectory and tool-call grading ideas from the junior eval rung (mod-205 in the Agentic AI Engineer track). The new requirement is **packaging**: the harness must be agent-agnostic. Use any model provider; you may stub the agents with fixed trajectories so the exercise is about the harness, not the agents.

## Prerequisites

- An agent loop that can emit a structured `Trajectory` (ordered steps: reason / tool_call / tool_result / answer).
- `async` Python and `pydantic` (or dataclasses).
- An API key in an environment variable for the LLM-judge grader; a small spend cap.

## Tasks

### 1. Define the grader interface

- Define `GradeResult` (name, score `0.0–1.0`, passed, detail) and a `Grader` protocol with a single `grade(case, trajectory) -> GradeResult` method.
- Every grader you write must conform to this protocol — no special cases.

### 2. Implement three grader families

- **Trajectory grader:** in-order-subset check that the agent's tool calls contain the case's `expected_tools` in order, plus a step-budget check. Reference-free where no expectation is given.
- **Tool-call grader:** precision/recall over the set of tools used vs. `expected_tools`, returned as a normalized score.
- **LLM-judge grader:** score the final answer against a rubric (faithfulness + no hallucinated citation) with a pinned judge model and temperature. Wrap the judge call so a failure degrades to `passed=False` with a detail, never an exception that crashes the run.

### 3. Load a versioned dataset

- Define an `EvalCase` schema and load cases from a JSONL file with a `dataset_version` recorded alongside.
- The harness must record which dataset version a run used in its report.

### 4. Run the harness and emit a report

- Implement `Harness.run(agent_fn, cases, graders) -> EvalReport`, where `agent_fn(task) -> Trajectory` is the only thing an agent must provide.
- `EvalReport` carries a `run_id`, the dataset version, per-case `GradeResult`s, and aggregate score per grader. Make it JSON-serializable.
- Fail soft: one case raising records a failed case and continues; the run still produces a report.

### 5. Prove reusability

- Run the *same* harness, graders, and dataset against **two** different `agent_fn`s (e.g., a "good" agent and a deliberately broken one that skips a required tool).
- Show the reports differ in the expected graders and that you changed **zero** harness code to evaluate the second agent.

## Starter guidance

```python
from dataclasses import dataclass, field
from typing import Protocol, Callable, Awaitable
import json, uuid

@dataclass
class GradeResult:
    name: str
    score: float
    passed: bool
    detail: str = ""

class Grader(Protocol):
    name: str
    def grade(self, case: "EvalCase", trajectory) -> GradeResult: ...

@dataclass
class EvalCase:
    id: str
    task: str
    expected_tools: list[str] | None = None
    reference: str | None = None

@dataclass
class EvalReport:
    run_id: str
    dataset_version: str
    per_case: dict = field(default_factory=dict)     # case_id -> list[GradeResult]
    aggregates: dict = field(default_factory=dict)    # grader_name -> mean score

def load_cases(path: str) -> tuple[list[EvalCase], str]:
    raise NotImplementedError  # read JSONL + dataset_version

class Harness:
    def __init__(self, graders: list[Grader]):
        self.graders = graders
    async def run(self, agent_fn: Callable[[str], Awaitable], cases, version) -> EvalReport:
        raise NotImplementedError  # run agent_fn per case, apply graders, fail soft
```

You do **not** need OTel tracing (exercise-02) or a CI gate (exercise-03) here — just the harness and report.

## Acceptance criteria

You can demonstrate that:

- Every grader conforms to the one `Grader` protocol; the harness applies them uniformly.
- All three grader families run and return normalized `0.0–1.0` scores.
- Cases load from a JSONL file and the report records the `dataset_version`.
- The `EvalReport` is JSON-serializable and carries per-case results plus per-grader aggregates.
- One case raising an exception does **not** crash the run; the report records it as failed.
- The **same** harness evaluates **two** different agents with no harness code changes, and the reports differ as expected.

## Reflection

In `NOTES.md`:

1. Where did you draw the agent-agnostic boundary, and what would break it if a team tried to "just add one field" to your harness?
2. Your LLM-judge grader is the least deterministic. What did you pin, and how would you measure its run-to-run variance before trusting it in a gate?
3. If two teams disagree on a step budget, does that belong in the harness or in their config? Why?

## Stretch goals

- Add a `GraderConfig` so teams compose graders and thresholds from a YAML/JSON file without touching code.
- Add aggregate precision/recall for the tool-call grader *across the whole dataset*, not just per case.
- Make the harness stream results so a 500-case run reports progress instead of blocking until done.
