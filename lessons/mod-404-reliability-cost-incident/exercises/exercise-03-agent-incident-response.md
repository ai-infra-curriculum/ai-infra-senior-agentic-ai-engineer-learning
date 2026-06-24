# exercise-03: Agent Incident Response

**Estimated effort:** 4 hours

## Objective

Run a simulated agent incident from detection to postmortem. You'll trigger a runaway-loop failure, detect it via your SLIs, take command, contain it with pre-built levers, recover, and write a blameless postmortem with tracked, typed action items. The point is to exercise the *whole loop* once, with a real timeline and real evidence, so the runbook isn't theoretical the night it matters.

## Background

This exercise covers material from:

- [Chapter 3 — Incident Response for Agent Failures](../03-incident-response-for-agents.md)
- [Chapter 4 — Postmortems and Durable Fixes](../04-postmortems-and-durable-fixes.md)

Build on exercise-01 (SLIs/alerts) and exercise-02 (budgets/breaker). This exercise is mostly **operational** — your deliverables are a timeline, a runbook, and a postmortem, not a new library.

## Prerequisites

- The SLIs and alerts from exercise-01 and the cost controls from exercise-02 wired into a runnable agent.
- A way to inject a fault: a prompt/tool change that makes the agent oscillate between two tool calls (a runaway loop).
- A scratch incident channel or doc where you can keep a timestamped log.

## Tasks

### 1. Pre-build the containment levers

- A **kill switch / feature flag** to disable the offending agent or tool path.
- Config-driven **budget tightening** (from exercise-02) you can change without a deploy.
- A documented **rollback** path for prompt/model/tool changes ("what deployed in the last 24h?").

### 2. Trigger and detect

- Inject the runaway-loop fault and let traffic flow.
- Confirm your **SLI alert** (loop-health burn-rate from exercise-01) and **cost-SLI** spike fire. Record the detection time and source.
- If detection is slow or silent, fix the alert first — silent detection is itself an incident finding.

### 3. Run the incident

- **Declare** and assign roles: an incident commander (decides, doesn't type fixes) and a scribe (keeps the timestamped log). Declare early.
- **Triage:** pull traces for the spiking runs; identify the oscillating tool calls.
- **Correlate:** answer "what changed?" and match the timeline to your injected fault.
- **Contain:** apply the fastest lever (usually rollback), plus a belt-and-suspenders budget tightening.
- **Verify and stand down:** confirm SLIs recover; hold open through one SLI window before closing.

### 4. Write the postmortem

- Use the template in Chapter 4. Keep it **blameless** — every root cause is a system statement, not a person.
- Drive root cause with **five whys** to a fix you can build and test (not "the model should behave better").
- Produce action items that are **owned, dated, ticketed, and typed** (prevent / detect / mitigate), with at least one of each type.

## Starter guidance

This deliverable is a runbook and a postmortem, not Python. Use these templates.

**Incident runbook (fill in during the drill):**

```markdown
# Incident: runaway-loop drill

- Detected: <UTC time> via <SLI alert | cost spike | report>
- Commander: <name> · Scribe: <name>
- Failure class: runaway-loop · Severity: <SEV-n>

## Containment checklist
- [ ] Pull traces for spiking runs; confirm the loop signature
- [ ] "What changed in last 24h?" -> <deploy/prompt/tool>
- [ ] Roll back the correlated change
- [ ] Tighten per-run step/token cap via config (no deploy)
- [ ] Confirm loop-health + cost SLIs recovering
- [ ] Hold open one SLI window, then stand down

## Timeline (UTC)
- HH:MM — <event>
```

**Postmortem skeleton (from Chapter 4):**

```markdown
# Postmortem: runaway-loop in <agent>
- Date / duration · Severity · Failure class: runaway-loop
- Impact: users affected, extra USD burned (from cost SLI), side-effects
- Timeline (UTC): from the scribe's log
- Detection: how found, what the SLI should have caught and when
- Root cause: five-whys to a SYSTEM cause; correlated change
- Action items table: | item | prevent/detect/mitigate | owner | due | ticket |
```

## Acceptance criteria

You can demonstrate that:

- Containment levers (kill switch, config budget, rollback) existed **before** the drill and were usable without a code deploy.
- The incident was **detected by an SLI/cost alert** (not only by you watching), with a recorded detection time and source.
- The incident ran with a named commander and a scribe, and produced a timestamped timeline from declare to stand-down.
- The postmortem is blameless, reaches a **system** root cause via five whys, and lists owned/dated/ticketed action items spanning prevent, detect, and mitigate.

## Reflection

In `NOTES.md`:

1. What was your time-to-detect and time-to-resolve? Which control shortened each the most?
2. Did any containment lever require a deploy? If so, that's an action item — what is it?
3. Rewrite one "blamey" root cause you were tempted to write as a blameless system statement.

## Stretch goals

- Run a **tool-misuse** drill instead (an agent calling a destructive tool with wrong arguments) and compare containment: note where irreversible side-effects change the first move.
- Add a **bad-output** drill (a faithfulness regression) where detection is hardest, and measure how long it takes your SLIs to catch it.
- Track your action items to completion and re-run the same drill; show the system is measurably harder to break that way.
