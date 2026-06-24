# Chapter 4 — Scoping and Sequencing Delivery

"We're going to build an agentic system that automates the support tier." That's an *initiative*, not a plan. Left unscoped, it becomes a six-month effort that integrates everything at once, proves nothing until the end, and either ships late or ships a thing nobody validated. Your job as the tech lead is to turn that fog into a **sequence** — a series of slices that each ship something real, de-risk the unknowns first, and survive contact with reality.

Agentic initiatives are *unusually* prone to this failure because the core uncertainty isn't "can we build it" — it's "will the model actually do the task reliably enough." You don't know that until you measure it, and you can't measure it from a Gantt chart.

## Scope: cut the initiative down to its spine

Before you sequence anything, find the **smallest slice that proves the core risk**. For an agentic initiative the core risk is almost always *task reliability*: does the agent do the job well enough, often enough, safely enough?

- **Name the riskiest assumption.** "The agent can resolve a support ticket end-to-end without a human" is the bet the whole initiative rests on. If that's false, no amount of polished UI matters.
- **Find the thinnest end-to-end slice that tests it.** One ticket category, one tool, a measurable success rate on a held-out eval set. Not the full system — the *spine* that proves or kills the assumption.
- **Be willing to descope to a copilot.** If "fully autonomous" is unproven, the first shippable slice might be "agent drafts, human approves." That ships value *and* generates the data to evaluate autonomy later. Descoping ambition to ship-and-measure is a feature, not a retreat.

## Sequence: de-risk first, value early, integrate last

Order the work so the scary, uncertain parts come *first* and each slice ships something. Three principles, often in tension — your job is to balance them:

- **De-risk first.** Sequence the riskiest, least-understood work earliest, while you still have time and budget to change course. If reliability of the core agent loop is the big unknown, prototype and *evaluate* that in week one — not after you've built the UI, the integrations, and the dashboards around an agent that turns out not to work.
- **Value early.** Each slice should ship something a user or stakeholder can actually use or react to. "Agent drafts replies for human approval" is shippable and generates real-world signal; "we built the memory layer" is not. Early value buys trust and budget for the rest.
- **Integrate incrementally, not in a big bang.** Avoid the plan where nothing works until everything is connected in the final week. Build against stubs and fake components (Chapter 3), integrate piece by piece, and keep the system runnable at every step.

When de-risk-first and value-early conflict — the riskiest thing isn't the most visible thing — usually **de-risk wins early**, because shipping visible value on top of an unproven core just builds more to throw away. But make that trade-off explicitly and tell stakeholders why slice one is "boring."

## A scoping-and-sequencing framework

A repeatable pass for any agentic initiative:

1. **State the outcome and the core bet.** One sentence each: what does success look like, and what's the riskiest assumption it rests on?
2. **Define "done enough" measurably.** What eval / success rate / safety bar makes this real? If you can't measure it, you can't ship it responsibly.
3. **Cut the spine slice.** The thinnest end-to-end path that tests the core bet against that measure.
4. **Order remaining slices by risk, then value.** Riskiest-unknown first; within similar risk, most-valuable first. Each slice ships and stays runnable.
5. **Mark decision points.** After which slice do you decide go / pivot / kill? Name them now so the initiative can change course on evidence, not sunk cost.
6. **Sequence with interfaces and parallelism.** Where can slices run in parallel behind frozen contracts (Chapter 3)? Where's the hard dependency chain?

## Communicating the plan

A delivery plan is also a leadership artifact:

- **Make the sequence and its logic legible.** Stakeholders should see *why* slice one is the unglamorous reliability prototype and not the demo-friendly UI. "We prove it works before we build around it" is a story that earns patience.
- **Tie slices to decision points.** "After slice 2 we'll have a measured success rate; if it's below X we pivot to copilot mode." This turns an open-ended bet into a managed one.
- **Keep it honest about uncertainty.** Agentic reliability is genuinely hard to predict. A plan that pretends otherwise loses credibility the first time the model underperforms. Name the risk; show how the sequence manages it.

## Key takeaways

- Scope an initiative down to its **spine** — the thinnest end-to-end slice that proves the riskiest assumption, which for agents is almost always **task reliability**, measured against an eval bar.
- Sequence to **de-risk first, ship value early, integrate incrementally**; when they conflict, de-risk usually wins early — but say so explicitly.
- Use the framework: outcome + core bet → measurable "done enough" → spine slice → order by risk then value → **named decision points** → interface-bounded parallelism.
- A delivery plan is a **leadership artifact**: make the sequencing logic legible, tie slices to go/pivot/kill decisions, and stay honest about reliability uncertainty.
