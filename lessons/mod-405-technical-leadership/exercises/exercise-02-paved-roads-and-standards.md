# exercise-02: Paved Roads and Standards

**Estimated effort:** 3 hours

## Objective

Write a paved-road RFC and a golden-path template for a common agent-building task, designed so the team actually adopts it — because the road is easier than going off-road, and the safety properties are baked into the default. By the end you'll have an RFC and a template that would let an engineer ship a PR that already passes most of the exercise-01 review rubric.

## Background

This exercise covers material from:

- [Chapter 2 — Mentoring and Paved Roads](../02-mentoring-and-paved-roads.md)
- [Chapter 1 — Leading Code Review for Agent Systems](../01-agent-code-review.md) — the standards your road bakes in.

A paved road is the supported, opinionated, *easy* path for a recurring task. You'll write one for "adding a new worker agent" (or another recurring task in your codebase).

## Prerequisites

- The agent-PR review rubric from [exercise-01](exercise-01-agent-code-review-standards.md) — your paved road should make most of its rows pass by default.
- A recurring agent-building task in your team's work (adding a worker, registering a tool, wiring an eval).

## Tasks

### 1. Pick the task and find the friction

- Choose one recurring task. Write the current "how do I do this?" path as an engineer experiences it today — every guess, every place they could get the safety properties wrong.
- Each friction point or easy-to-get-wrong property is a reason the paved road exists.

### 2. Write the paved-road RFC

- Fill in the RFC template in Starter guidance. The core sections: the problem, the proposed default path, what it bakes in (tie each to a rubric row), what going off-road costs, and how you'll roll it out.
- For every standard the road enforces, write the one-line *why*. Engineers route around standards they don't understand.

### 3. Build the golden-path template

- Produce the actual template artifact (a directory skeleton, a generator, or a documented example) with the safe defaults already wired: bounded loop, scoped+audited tool registration, trust boundary on untrusted input, a starter eval case.
- Include a short README that says what to change, what *not* to change, and why.

### 4. Make adoption easy

- Write the one thing that makes the road faster than going off-road (a generator command, a copy-this-folder instruction, an `npm create` / cookiecutter, etc.). If using the road is slower than not, it won't be used.

## Starter guidance

Use this RFC template as your starting artifact.

```markdown
# RFC: Paved road for <recurring task>

## Status
Draft | Proposed | Adopted

## Problem
<The recurring task, and the safety/consistency properties that are easy to get
wrong today. Cite the friction an engineer hits now.>

## Proposed paved road
<The default, opinionated, easy path. Link the template/generator.>

## What it bakes in
| Default | Rubric row it satisfies | Why (one line) |
|---------|-------------------------|----------------|
| Bounded loop wired in | Loop & resource bounds | runaway loops are cost incidents |
| Tools via scoped+audited registry | Tool authority | wrong model call = real side effect |
| Trust boundary on untrusted input | Prompt-injection surface | retrieved text can carry instructions |
| Starter eval case required | Evaluation | behavior change needs an eval, not a unit test |

## Going off-road
<When it's allowed, and what extra review/justification the engineer owns if they
leave the road. The road is easier, not compulsory.>

## Rollout
<How engineers discover and use it: generator command, docs link, example PR.
What makes the road faster than not using it.>

## Non-goals
<What this road deliberately does NOT cover, so scope stays small and living.>
```

Golden-path template skeleton (adapt to your stack):

```text
worker_template/
  worker.<ext>        # loop with hard max-iterations bound already present
  tools.<ext>         # tools registered through scoped, audited registry
  prompts/system.md   # system prompt with injection-resistance preamble
  eval/cases.yaml     # one starter eval case; PR must add real ones
  README.md           # what to change, what NOT to change, and why
```

## Acceptance criteria

You can demonstrate that:

- Your RFC names the real friction and proposed default path, with a one-line *why* for every standard it enforces.
- The "what it bakes in" table maps each default to an exercise-01 rubric row.
- Your golden-path template exists as a concrete artifact with the safe defaults actually wired (not described — present).
- An engineer copying the template would produce a PR that passes the loop-bound, tool-scoping, trust-boundary, and eval rows of the rubric by default.
- You stated what going off-road costs and what makes the road faster than not using it.

## Reflection

In `NOTES.md`:

1. Which baked-in default removes the most review burden — i.e., which rubric finding will you now almost never have to write by hand?
2. A senior engineer wants to go off-road for a real reason. How does your standard handle that without becoming either a wall or a rubber stamp?
3. How will you know in three months whether the road was adopted? Name the signal (template usage, drop in a class of review finding, etc.).

## Stretch goals

- Convert the template into a real generator (`cookiecutter`, a CLI command, a repo template) and use it to scaffold a working worker.
- Add a CI check that fails a PR which builds a worker *without* the paved-road defaults (e.g., an unbounded loop), nudging engineers onto the road.
- Write the rollout announcement you'd send the team — the version that makes them *want* to use the road, not the version that mandates it.
