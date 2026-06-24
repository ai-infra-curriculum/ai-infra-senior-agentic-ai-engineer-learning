# Chapter 2 — Implementation-Level Tradeoffs

The architecture left three big decisions inside your box: which framework runs the agent loop, where conversational and task state lives, and how wide the tool surface gets. None of these has a universally right answer. What separates a senior implementation is not the choice itself but that the choice was *made* — weighed against alternatives, fit to this subsystem's actual constraints, and written down so the next person inherits the reasoning, not just the result.

The artifact for that is an **Architecture Decision Record (ADR)**: a short, dated note capturing the decision, the options considered, the forces that drove it, and the consequences you accept. Treat the rest of this chapter as the three ADRs you will most often have to write.

## Framework choice: adopt, wrap, or hand-roll

The temptation is to pick the framework you know. The senior move is to pick the one whose *primitives match your subsystem's hard requirement*, and to wrap it so the choice is reversible.

| Force | Favors a framework (LangGraph, Agents SDK) | Favors hand-rolling the loop |
| --- | --- | --- |
| Control flow is graph-shaped, with branches and resumable state | Strong — this is what they are for | Weak — you reimplement their state machine |
| You need fine control over the exact token stream and retry semantics | Weak — you fight the abstraction | Strong |
| Team must onboard fast and read familiar patterns | Strong | Weak |
| Subsystem is tiny (one tool call, one model call) | Weak — dependency cost exceeds benefit | Strong |
| You expect to swap providers or runtimes later | Neutral — wrap it either way | Neutral — wrap it either way |

The decisive rule: **wrap whatever you choose behind your own interface.** If your executor depends directly on a framework's `Runnable` type throughout, that framework is now load-bearing and a migration is a rewrite. If it depends on your `ToolExecutor` protocol and the framework lives behind one adapter, swapping it is a contained change. The framework is an implementation detail of your box; never let it leak into the contract.

## State management: where does the truth live?

Every agent subsystem holds state — conversation history, task progress, partial results, tool-call dedup keys. The tradeoff is **locality versus durability**, and getting it wrong is the most common production incident in agent systems.

- **In-memory / in-process state** is fast and simple, and it evaporates on restart. Fine for a single-request synthesizer; catastrophic for a long-running planner that must survive a deploy.
- **External durable state** (Postgres, Redis, a checkpoint store) survives restarts and lets you resume a paused agent, at the cost of serialization, schema migration, and a network hop on every step.

Three rules keep this decision honest:

1. **Separate ephemeral from durable explicitly.** A scratchpad the model uses within one turn does not belong in Postgres. The agreed task state that another subsystem may read does. Mixing them produces both slow trivial operations and lost important ones.
2. **Make durable state the single source of truth, and derive the rest.** Per the immutability principle, do not mutate a shared state object in place across steps — append a new versioned record and let readers project the current view. This is what makes a paused agent resumable and a bug reproducible.
3. **Decide idempotency at the state layer.** If your executor is required to be idempotent on `call.id`, that key lives in durable state. "Idempotent" is a storage decision, not a prompt instruction.

```python
# Ephemeral: lives and dies with the turn.
scratch: dict = {}

# Durable: the contract's source of truth, append-only, keyed for idempotency.
async def record_result(store, call_id: str, result: ToolResult) -> None:
    existing = await store.get(call_id)
    if existing is not None:
        return  # idempotent: a replay is a no-op
    await store.put(call_id, result)  # new record, never mutate in place
```

## Tool boundaries: how wide is the surface?

Each tool you expose to the agent is attack surface, failure surface, and prompt-budget surface at once. The tradeoff is **capability versus controllability**.

- A **broad tool** (`run_sql(query)`) is flexible and almost impossible to constrain — the model can do anything the database permits, and you cannot reason about what it will do.
- A **narrow tool** (`get_customer_orders(customer_id)`) is trivial to validate, authorize, rate-limit, and test, at the cost of needing more tools to cover the same ground.

For a production subsystem, draw boundaries narrow by default and widen only with evidence. A narrow tool gives you a chokepoint to enforce input validation, scope authorization (this agent may read orders, not write them), and per-tool rate limits — the same boundary-validation discipline you apply to any external input. A broad tool gives you a single `// validate everything here` comment that no one ever fully implements. When you must expose something powerful, put the policy in the tool's executor, not in the system prompt; a prompt is a suggestion, an executor check is a guarantee.

## Writing the decision down

For each of the three decisions, the ADR is four lines that save the next engineer a week:

```text
# ADR-012: State store for the planner subsystem
Status:   Accepted (2026-06-23)
Context:  Planner must resume after deploys; tasks run up to 20 min.
Decision: Durable checkpoints in Postgres, keyed by task_id; scratch stays in-process.
Consequence: +1 network hop per step; gain resumability and reproducible bug reports.
            Revisit if step latency budget drops below 50 ms.
```

The ADR is not bureaucracy. It is how a tradeoff survives the person who made it — and how you defend the choice when an architect asks "why Postgres and not Redis?" six months from now.

## Key takeaways

- Implementation tradeoffs are graded on whether they were *made deliberately and documented*, not on landing the objectively best option.
- Wrap any framework behind your own interface so the choice stays reversible and never leaks into the contract.
- Split state into ephemeral and durable, make durable state an append-only source of truth, and treat idempotency as a storage decision.
- Draw tool boundaries narrow by default; enforce validation, authorization, and limits in the executor, never in the prompt.
- Capture each decision as a dated ADR — the four lines that let a tradeoff outlive you.
