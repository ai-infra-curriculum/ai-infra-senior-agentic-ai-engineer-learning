# Chapter 3 — Durable Execution and Resumption

A long-running agent workload — a research run that fans out to thirty workers over ten minutes, a multi-step pipeline that calls a dozen tools — is a bet that nothing crashes for the whole duration. At scale that bet loses constantly. A deploy restarts the process. A rate-limit pauses you mid-run. A node gets preempted. Without durability, every one of those events throws away minutes of work and money and starts from zero. Durable execution makes the workload **resume** from where it stopped instead.

## Why agents need this more than most services

A stateless web request is cheap to retry — re-run it and you're done. An agent run is the opposite:

- **It's long.** Minutes, not milliseconds. The probability of *something* interrupting it grows with duration.
- **It's expensive.** Restarting from zero re-pays for every token already spent (Chapter 2). A crash at minute nine of a ten-minute run isn't a small loss.
- **It has side effects.** Workers send emails, write to databases, file tickets. Naively retrying the whole run can send the same email twice. Resumption has to be careful, not just "run it again."

Durability turns a long, expensive, side-effecting workload into a sequence of small, recoverable steps.

## Checkpoint the state, not the process

The core technique is simple: persist progress to durable storage after each meaningful step, so a fresh process can load it and continue. Model the workflow as explicit state — completed steps, pending steps, and accumulated results — and write it down:

```python
@dataclass
class WorkflowState:
    run_id: str
    pending: list[str]            # assignments not yet done
    completed: dict[str, dict]    # assignment_id -> distilled result
    status: str = "running"       # running | done | failed

async def resume_or_start(run_id: str, assignments: list[str]) -> WorkflowState:
    state = await store.load(run_id)
    if state is None:
        state = WorkflowState(run_id, pending=list(assignments), completed={})
        await store.save(state)
    return state

async def run(state: WorkflowState):
    for aid in list(state.pending):
        result = await run_worker(aid)        # the expensive step
        state.completed[aid] = result
        state.pending.remove(aid)
        await store.save(state)               # checkpoint after each step
    state.status = "done"
    await store.save(state)
```

If the process dies after worker 7, restarting with the same `run_id` reloads the state, sees 7 done and 23 pending, and continues. Granularity is the trade-off: checkpoint too coarsely and a crash loses a lot of work; checkpoint too finely and you pay storage overhead on every tiny step. Checkpoint at the boundaries where redoing the work is expensive — typically after each model call or tool result.

## Idempotency: the rule that makes resumption safe

Resumption means a step might run more than once: the process could crash *after* doing the work but *before* recording it. So every step with a side effect must be **idempotent** — safe to execute twice with the same effect as once.

- **Tag external actions with an idempotency key.** Derive a stable key (e.g., `run_id + step_id`) and pass it to APIs that support deduplication, or check "did I already do this?" before acting.
- **Make writes upsert, not append.** Writing `completed[aid] = result` twice is harmless; appending to a list twice duplicates.
- **Quarantine truly non-idempotent actions.** If an action genuinely can't be made safe to repeat (a payment with no idempotency support), record intent durably *before* acting and confirm *after*, so resumption can check status instead of re-firing.

Idempotency is what separates "resume" from "duplicate." It is not optional at scale.

## Deterministic replay vs. checkpointed state

There are two families of durable execution, and it's worth knowing which you're using.

- **Checkpointed state** (above) stores the *results* of each step. Resumption reads results and skips completed work. Simple, explicit, and language-agnostic.
- **Deterministic replay** (the model behind engines like Temporal) stores an *event history* of every step's input and output, then re-executes your workflow code from the top — but instead of re-running a completed step, it returns the recorded result from history. Your orchestration logic reads like a straight-line function; the engine makes it durable underneath.

Replay is powerful but imposes a discipline: the workflow code must be **deterministic**. Anything non-deterministic — wall-clock time, random numbers, direct network calls, and notably the model's own stochastic output — must be pushed into recorded "activities," not run inline, or the replay diverges from history. For agents this is a real constraint: the LLM call itself is non-deterministic, so it *must* live in an activity whose result is recorded. Get that boundary right and you get crash-proof, rate-limit-proof orchestration almost for free.

## Resumption in practice

Production resumption ties together the threads from earlier chapters:

- **Survive rate-limits as pauses, not failures.** When a provider returns `429`, a durable workflow can sleep with backoff (Chapter 4) and resume — the run is paused, not lost.
- **Survive deploys.** Checkpointed runs reattach after a rolling restart instead of all failing at once.
- **Bound retries.** A step that fails repeatedly shouldn't retry forever; cap attempts and route the run to a dead-letter path (Chapter 4) for inspection.
- **Expose run status.** A durable run has a queryable state — `running`, `completed`, `failed`, and progress within it — which is also exactly what an operator needs during an incident.

## Key takeaways

- Agent runs are long, expensive, and side-effecting — exactly the workloads where restarting from zero is unacceptable.
- Checkpoint **state** (completed steps + results) to durable storage after each expensive step so a fresh process resumes instead of restarting.
- Every side-effecting step must be **idempotent** (idempotency keys, upserts, intent-then-confirm) — this is what makes resumption safe rather than duplicating work.
- Two models: explicit **checkpointed state** (simple, language-agnostic) vs. **deterministic replay** (Temporal-style) — replay needs non-determinism, including the LLM call, pushed into recorded activities.
- Durability turns rate-limits and deploys into resumable pauses, and gives operators a queryable run status for free.
