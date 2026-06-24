# Chapter 4 — Postmortems and Durable Fixes

An incident you contain but don't learn from is an incident you'll have again. The postmortem is the mechanism that converts a 2am fire into a durable fix and a smarter system. The SRE practice here is mature — Google's postmortem culture rests on two pillars: it is **blameless**, and it produces **tracked action items**. Both are easy to say and hard to hold. This chapter is about holding them for agent incidents specifically, where root causes are often probabilistic and the temptation to "just tweak the prompt" is strong.

## Blameless means systems, not people

The blameless principle: assume everyone acted reasonably with the information they had, and ask *what about the system* let a reasonable action cause an incident. "The engineer shipped a bad prompt" is a dead end — it produces fear and no fix. "A prompt change reached production with no eval gate on faithfulness, so a regression was invisible until users hit it" is a systems statement — it points at a missing control you can build.

Blameless is not consequence-free; it's **cause-focused**. The output of a good postmortem is a list of system changes, not a list of people who should be more careful. People are already careful; careful people still cause incidents when the system permits it.

## Root cause without stopping at the prompt

For agent incidents, the lazy root cause is "the model hallucinated" or "the prompt was ambiguous." Those are *symptoms*. Use the **five whys**, but aim each "why" at the system:

> Bad outputs reached users.
> *Why?* The agent cited a fact not in the retrieved context.
> *Why did that reach users?* No faithfulness check ran before responding.
> *Why was there no check?* The grounding SLI existed but wasn't wired as a release gate.
> *Why wasn't it a gate?* We treated it as a dashboard metric, not a budget.
> **Durable fix:** make grounding a release-gating SLO (Chapter 1), and add a runtime faithfulness check that degrades to human review on low scores (Chapter 3).

The discipline is to keep going until the fix is a **system change you can build and test**, not a request for the model to behave better. A probabilistic component will always fail sometimes; the fix is the guardrail around it, not a promise from it.

## Action items that actually ship

The most common way a postmortem fails is a list of good intentions with no owner and no due date. Every action item needs four things:

- An **owner** (a person, not a team).
- A **due date**.
- A **tracking link** (a ticket in the normal backlog, so it competes for time honestly).
- A **type:** *prevent* (stop this class recurring), *detect* (catch it faster next time), or *mitigate* (reduce blast radius when it does).

A balanced postmortem has items across all three types. For the example above: *prevent* = the release-gating SLO; *detect* = a baseline-drift alert on grounding; *mitigate* = the degrade-to-human path. If every action item is "prevent," you have no faster detection next time; if every item is "detect," you keep catching the same fire.

## A postmortem template for agent incidents

Use a consistent template so postmortems are quick to write and easy to read across a team. The agent-specific additions to the standard SRE template are the **failure class**, the **change correlation**, and the **cost impact**.

```markdown
# Postmortem: <short title>

- **Date / duration:** <when, and time-to-detect / time-to-resolve>
- **Severity:** <SEV-n> · **Failure class:** runaway-loop | tool-misuse | bad-output
- **Author / reviewers:** <names> · **Status:** draft | reviewed

## Impact
- User-facing: <who saw what, how many>
- Cost: <extra USD burned, from the cost SLI / attribution>
- Side-effects: <any irreversible tool actions, e.g. records changed>

## Timeline (UTC)
- HH:MM — <event, from the scribe's incident log>

## Detection
- How we found out: SLI alert | cost spike | user report
- What the SLI should have caught and when: <gap analysis>

## Root cause
- Five-whys to a SYSTEM cause (not "the model hallucinated").
- Correlated change: <prompt/model/tool deploy, if any>

## What went well / what was hard
- <kept blameless; name the controls that worked, e.g. per-run cap bounded it>

## Action items
| Item | Type (prevent/detect/mitigate) | Owner | Due | Ticket |
|------|-------------------------------|-------|-----|--------|
| ...  | ...                           | ...   | ... | ...    |
```

## Close the loop

A postmortem isn't done when it's written; it's done when its action items ship and the underlying SLO is healthier. Review postmortems in a regular forum, track action-item completion as its own metric (stale postmortem items are a reliability smell), and feed the durable fixes back into the system: a new release-gating SLO, a new containment lever, a tightened budget. Done well, each incident leaves the system measurably harder to break in that way again — which is the entire point of the discipline.

## Key takeaways

- Postmortems are **blameless** (systems, not people) and **action-driven** (owned, dated, tracked items) — both pillars are required.
- Push root cause past "the model hallucinated" with **five whys aimed at the system**; stop only at a fix you can build and test.
- Every action item has an **owner, due date, ticket, and type**; balance **prevent / detect / mitigate** so you don't just re-prevent.
- The loop closes when action items **ship and the SLO improves** — each incident should leave the system harder to break that way again.
