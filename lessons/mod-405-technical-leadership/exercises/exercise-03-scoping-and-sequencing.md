# exercise-03: Scoping and Sequencing

**Estimated effort:** 3 hours

## Objective

Turn a vague agentic initiative into a sequenced delivery plan: scope it down to the spine slice that proves the riskiest assumption, sequence the rest to de-risk first and ship value early, and mark the decision points that let the initiative pivot on evidence instead of sunk cost. By the end you'll have a one-page delivery plan a stakeholder could read and a team could execute.

## Background

This exercise covers material from:

- [Chapter 4 — Scoping and Sequencing Delivery](../04-scoping-and-sequencing.md)
- [Chapter 3 — Translating Design to Execution](../03-design-to-execution.md) — interfaces and parallelism in the sequence.

## Prerequisites

- An agentic initiative — a real one from your work, or the sample below: *"Build an agentic system that automates tier-1 customer support."*
- The mindset from [mod-404](../../mod-404-reliability-cost-incident/README.md) that "done enough" for an agent is a measured reliability/safety bar, not a demo.

## Tasks

### 1. State the outcome and the core bet

- In one sentence each: what does success look like, and what's the single riskiest assumption the whole initiative rests on? For an agentic initiative this is almost always a *task-reliability* assumption.

### 2. Define "done enough" measurably

- Write the eval / success-rate / safety bar that makes the core slice real. If you can't state how you'd measure it, you can't ship it responsibly — fix that first.

### 3. Cut the spine slice

- Define the thinnest end-to-end slice that tests the core bet against that measure. Resist building the whole system. Be willing to descope autonomy to a copilot ("agent drafts, human approves") if full autonomy is unproven.

### 4. Sequence the remaining slices

- Order them: riskiest-unknown first, then most-valuable. Each slice must ship something usable and keep the system runnable (integrate incrementally, not big-bang).
- Note where slices can run in parallel behind frozen interfaces, and where there's a hard dependency chain.

### 5. Mark decision points

- After which slice do you decide go / pivot / kill, and on what measured signal? Name them now.

### 6. Write the one-page plan

- Fill in the delivery-plan template in Starter guidance. It should be legible to a stakeholder: *why* slice one is the unglamorous reliability prototype is part of the story.

## Starter guidance

Use this delivery-plan template as your starting artifact.

```markdown
# Delivery plan: <initiative>

## Outcome
<One sentence: what success looks like.>

## Core bet (riskiest assumption)
<One sentence: the assumption the whole initiative rests on.>

## "Done enough" bar
<The measurable threshold: eval pass rate / success rate / safety bound.>

## Spine slice (proves the core bet)
| Slice | What ships | Proves / measures | Est. |
|-------|-----------|-------------------|------|
| 0 | <thinnest end-to-end path> | <core bet vs. the bar> | <t> |

## Sequence (de-risk first, value early, integrate incrementally)
| # | Slice | Ships value to | Risk it retires | Depends on | Parallel? |
|---|-------|----------------|-----------------|------------|-----------|
| 1 | | | | | |
| 2 | | | | | |
| 3 | | | | | |

## Decision points
| After slice | Signal measured | Go | Pivot | Kill |
|-------------|-----------------|----|----|----|
| | | | | |

## Why this order
<2-3 sentences a stakeholder reads: why slice 0 is "boring" reliability work and
not the demo-friendly UI; how the sequence manages the reliability uncertainty.>
```

Sample initiative to plan (use if you don't have your own): *"Automate tier-1
customer support with an agent."* Stakeholders imagine full autonomy. The core
bet is unproven: can the agent resolve a ticket category end-to-end at an
acceptable success and safety rate?

## Acceptance criteria

You can demonstrate that:

- Your plan states the outcome and the single riskiest assumption in one sentence each.
- "Done enough" is a measurable bar, not "it works in the demo."
- The spine slice is genuinely thin and end-to-end, and it tests the core bet against the bar (descoping to copilot mode if needed).
- The sequence puts the riskiest unknown early, ships usable value each slice, and integrates incrementally — with parallelism behind interfaces noted.
- At least one decision point ties a measured signal to a go / pivot / kill choice.
- The "why this order" paragraph would make a stakeholder accept a boring slice one.

## Reflection

In `NOTES.md`:

1. Where did de-risk-first conflict with ship-value-early in your plan, and how did you resolve it? Why?
2. What's the cheapest experiment that would kill the initiative fastest if the core bet is false? Is it slice 0? If not, why not?
3. Your stakeholder wants the flashy UI in slice 1. What's the one sentence you say to defend sequencing reliability first?

## Stretch goals

- Add a cost/budget column to the sequence and show how an early reliability result would change the spend you'd commit to later slices.
- Re-sequence the same initiative under a hard external deadline (a launch date) and explain what you'd cut and what you'd refuse to cut.
- Pair this plan with a translation pass from [exercise material in Chapter 3](../03-design-to-execution.md): pick one slice and write its interface contract so two engineers could build against it in parallel.
