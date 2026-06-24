# exercise-02: Durable Execution In Production

**Estimated effort:** 4 hours

## Objective

Make a long-running, side-effecting agent workflow survive a crash. You'll model the workflow as explicit, checkpointed state, kill the process mid-run, and prove it resumes from where it stopped instead of restarting from zero. Then you'll make the side-effecting steps **idempotent** so resumption doesn't duplicate work, and add bounded retries with a dead-letter path for steps that never succeed.

## Background

This exercise covers material from:

- [Chapter 3 — Durable Execution and Resumption](../03-durable-execution.md)
- [Chapter 4 — Hardening Inter-Agent Communication and Tools](../04-hardening-communication.md) (for retries and dead-lettering)

You can build this with a plain durable store (SQLite or even a JSON file with atomic writes) and your own checkpoint loop, *or* adopt a durable-execution engine (Temporal) and implement the same workflow with activities. Do at least the hand-rolled version first so you understand what the engine is doing for you.

## Prerequisites

- A multi-step workflow (e.g., the orchestrator-worker run from exercise-01, or a 10-step pipeline).
- A durable store you can read after a process restart (SQLite, a file with atomic rename, Redis, or Temporal).
- The ability to kill and restart your process (Ctrl-C, `kill`, or a crash injected with a deliberate exception).

## Tasks

### 1. Model the workflow as state

- Define an explicit `WorkflowState`: a `run_id`, the list of `pending` steps, a `completed` map of step results, and a `status`.
- Each step does a unit of expensive work (a stubbed worker call that sleeps, plus a *side effect* — append a line to an output file, or insert a row keyed by step id).

### 2. Checkpoint after each step

- After each step completes, persist the updated state to your durable store **before** moving on.
- Use an atomic write (write-temp-then-rename, or a DB transaction) so a crash mid-checkpoint can't corrupt the state.

### 3. Crash and resume

- Start a run with ~10 steps. Inject a crash after step 6 (raise inside the loop, or `kill -9` the process).
- Restart with the **same** `run_id`. The workflow must load state, see steps 1–6 done, and execute only 7–10. Steps 1–6 must **not** run again.

### 4. Idempotency

- First, demonstrate the hazard: crash *after* a step's side effect but *before* the checkpoint, so on resume that step re-runs. Show the duplicate (two lines / two rows).
- Then fix it: make the side effect idempotent — key the write on `run_id + step_id` and upsert, or check "already done?" before acting. Re-run the crash-after-effect scenario and show exactly one effect.

### 5. Bounded retries and dead-letter

- Make one step fail transiently (raise N times, then succeed). Add retry with exponential backoff + jitter, capped at a max attempt count.
- Make another step fail *permanently*. After exhausting retries, route it to a dead-letter store and mark the run `failed` (or continue with the step skipped and labeled) — without looping forever.

## Starter guidance

```python
import asyncio, json, os, random
from dataclasses import dataclass, field, asdict

@dataclass
class WorkflowState:
    run_id: str
    pending: list[str]
    completed: dict[str, dict] = field(default_factory=dict)
    status: str = "running"

def save(state: WorkflowState, path: str) -> None:
    tmp = f"{path}.tmp"
    with open(tmp, "w") as f:
        json.dump(asdict(state), f)
    os.replace(tmp, path)                      # atomic checkpoint

def load(path: str) -> WorkflowState | None:
    if not os.path.exists(path):
        return None
    with open(path) as f:
        return WorkflowState(**json.load(f))

async def do_step(run_id: str, step_id: str) -> dict:
    await asyncio.sleep(random.uniform(0.1, 0.5))   # expensive work
    side_effect(run_id, step_id)                    # task 4: must be idempotent
    return {"step_id": step_id, "ok": True}

async def run(state: WorkflowState, path: str):
    for sid in list(state.pending):
        state.completed[sid] = await with_retry(lambda: do_step(state.run_id, sid))
        state.pending.remove(sid)
        save(state, path)                          # checkpoint after each step
    state.status = "done"
    save(state, path)

async def with_retry(fn, attempts=4, base=0.3, cap=4.0):
    for i in range(attempts):
        try:
            return await fn()
        except Exception:
            if i == attempts - 1:
                raise
            await asyncio.sleep(min(cap, base * 2 ** i) + random.uniform(0, base))
```

## Acceptance criteria

You can demonstrate that:

- A run crashed after step 6 **resumes** on restart and runs only steps 7–10; completed steps do not re-execute.
- The checkpoint write is atomic — a crash during checkpointing leaves a valid (old or new) state, never a corrupt file.
- The idempotency hazard is shown (a duplicate side effect) **and** fixed (exactly one effect after the same crash-and-resume).
- A transiently failing step succeeds via bounded retry with backoff + jitter; a permanently failing step lands in a dead-letter store instead of looping forever.
- Run status is queryable: you can read `pending` / `completed` / `status` for any `run_id` at any time.

## Reflection

In `NOTES.md`:

1. At what granularity did you checkpoint, and what's the trade-off if you checkpoint more vs. less often?
2. Which steps were *not* naturally idempotent, and how did you make them safe to repeat?
3. If you also did the Temporal version: what did the engine handle for you, and what new constraint (determinism) did it impose? Where did the non-deterministic LLM call have to go?
4. How would resumption interact with a provider rate-limit — how does durability turn a `429` into a pause instead of a failure?

## Stretch goals

- Re-implement the workflow on **Temporal** (or another durable-execution engine) with the LLM/tool calls as activities, and confirm a worker restart resumes the workflow transparently.
- Add **resume-after-rate-limit**: on a simulated `429`, sleep-with-backoff inside the durable run and continue, so the run never fails — only pauses.
- Support **concurrent** steps within a checkpoint boundary (fan-out) while keeping resumption correct: a crash mid-fan-out must not double-run completed branches.
