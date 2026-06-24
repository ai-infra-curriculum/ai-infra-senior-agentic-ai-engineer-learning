# Chapter 3 — Translating Design to Execution

An architect hands you a design: a multi-agent system with an orchestrator, a set of specialist workers, a shared memory layer, and an evaluation harness. It's a good design. Now you have to turn it into something five engineers can build over the next quarter without it collapsing into a mess that no longer resembles the design — or grinding to a halt because the design left the hard questions unanswered.

This is the tech-lead translation layer: you sit between the design's *intent* and the team's *reality*, and your job is to preserve the first while respecting the second. Lose the intent and you've shipped something that isn't the system the architect designed. Ignore the reality and the team can't build it.

## A design doc is not a build plan

Architects describe **what the system is and why**: components, boundaries, the key trade-offs, the invariants that must hold. They deliberately leave out **how the team builds it and in what order** — that's your job, and it should be, because you know the team and the codebase in a way the design doc can't.

The gap between a design and a build plan is full of questions the design doesn't answer:

- **What's the build order?** Which component is the spine everything else hangs off, and which are leaves you can stub?
- **Where are the interfaces?** What contracts let two engineers work in parallel without blocking on each other?
- **What's genuinely uncertain?** Which parts of the design are "we know how to do this" versus "we *think* this works but haven't proven it"?
- **What can be stubbed or faked first?** Can you build the orchestrator against fake workers, or the workers against a fake orchestrator, to unblock parallel work?

## Preserve the intent, not the literal diagram

The design's value is its **intent** — the invariants and trade-offs that make it correct. Your translation must protect those even as the literal shape flexes against reality.

- **Find the load-bearing invariants and write them down.** "Workers never see each other's raw context" or "every tool call is audited" are the things that, if violated during execution, mean you built a different (worse) system. Make them explicit acceptance criteria, not tribal knowledge.
- **Distinguish intent from incidental detail.** The design says "use a Postgres-backed memory store." Is Postgres the *intent*, or is "durable, queryable shared memory" the intent and Postgres an example? Ask. Flexing incidental details is fine; flexing intent needs the architect back in the room.
- **Close the loop when reality bites.** When execution reveals the design can't be built as drawn — a contract is wrong, an assumption was false — that's not a failure, it's information. Take it *back* to the architect with the specifics. The worst outcome is the team silently working around the design until it no longer matches, and nobody decided to do that.

## Surfacing the unstated questions

Most of the value you add is asking the questions the design doc left implicit, *before* the team hits them at 2pm on a Wednesday blocked on each other:

- **Interface-first questions.** "The orchestrator calls workers — what's the exact request/response contract? Who defines it, and can we freeze it so two people build against it in parallel?" Nailing interfaces early is the single biggest unblocker for parallel work.
- **Uncertainty questions.** "The design assumes the evaluator can score outputs reliably. Have we proven that? If not, that's the first thing to prototype, because everything downstream depends on it." Pull the riskiest assumption to the front (see Chapter 4).
- **Reality questions.** "The design wants real-time memory writes, but our store has a 200ms write latency. Does the design tolerate eventual consistency, or do we need a different store?" Surface the mismatch *to the architect*, not as a silent workaround.

## Communicating in both directions

You translate *down* to the team and *up* to the architect, and both directions matter:

- **Down:** turn the design into concrete, sequenced, interface-bounded work with explicit acceptance criteria that encode the invariants. The team should be able to build without re-reading the whole design doc for every task.
- **Up:** feed reality back. When the team learns something that challenges the design, the architect needs it — fast, specific, and framed as "here's what we found, here's the decision I think we need," not "the design is broken."

## Key takeaways

- A **design doc is not a build plan**. The architect owns *what and why*; you own *how and in what order* — build order, interfaces, uncertainty, what to stub first.
- Preserve the design's **intent and load-bearing invariants** as explicit acceptance criteria; flex incidental details freely but take intent-level changes back to the architect.
- The value you add is **surfacing the unstated questions** — especially interface contracts and the riskiest assumptions — before the team hits them while blocked.
- Communicate in **both directions**: sequenced, interface-bounded work down to the team; specific, decision-framed reality up to the architect.
