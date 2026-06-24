# mod-404 Module Quiz — Reliability, Cost & Incident Response

## Instructions

- Covers all four chapters of mod-404.
- 12 multiple-choice questions plus 2 short-answer scenarios.
- Passing score: 80% (10/12 multiple choice).
- Answer key and explanations are at the end.

---

## Section A — SLOs and SLIs (Chapter 1)

### Question 1

Why is an availability SLI insufficient for an agentic system?

A) Availability is too expensive to measure
B) It passes responses that are fast and well-formed but wrong
C) Agents never return HTTP errors
D) SLIs don't apply to LLM systems

### Question 2

An SLI is best defined as:

A) A target percentage over a time window
B) The budget of allowable failures
C) A ratio of good events to valid events, measured continuously
D) An alert threshold

### Question 3

You compute a task-success SLI. Which subset of runs should the ratio be computed over?

A) All production runs, treating unlabeled runs as successes
B) All production runs, treating unlabeled runs as failures
C) Only the sampled, judged subset, reporting a confidence interval
D) Only the runs that returned an error

### Question 4

What does a burn-rate alert add over a single static threshold?

A) It removes the need for an error budget
B) It detects how fast the error budget is being consumed, catching both cliffs and slow slides
C) It only fires on a regression below baseline
D) It measures latency instead of quality

### Question 5

The most senior use of a quality error budget is to:

A) Display it on a dashboard
B) Gate releases — stop shipping prompt/model/tool changes when the budget is exhausted
C) Bill customers for failures
D) Replace incident response

## Section B — Cost Controls (Chapter 2)

### Question 6

Why is agent cost described as "unbounded by construction"?

A) Models have no per-token price
B) The agent decides at runtime how many model and tool calls to make
C) Cloud GPUs have no quota
D) Tokens are free below a threshold

### Question 7

In a per-run budget guard, when should you check the budget versus charge it?

A) Charge before, check after
B) Check and charge at the same time
C) Check before each call, charge after each call
D) Only check at the end of the run

### Question 8

A per-run cap cannot catch which failure?

A) A single request that loops 40 times
B) One tenant sending 10,000 well-behaved but expensive runs in an hour
C) A run that exceeds its token limit
D) A run that exceeds its step limit

### Question 9

When a per-tenant circuit breaker is open, the correct behavior is to:

A) Continue the run but log a warning
B) Reject fast with no model spend
C) Switch the tenant to a larger model
D) Silently drop the request with no metric

### Question 10

"Fail closed" for a cost guard means:

A) Continue serving at any cost to preserve availability
B) Stop and return a graceful bounded result rather than silently continuing to spend
C) Crash the process
D) Disable all logging

## Section C — Incident Response & Postmortems (Chapters 3–4)

### Question 11

Which mapping is correct for agent failure classes and their detection SLI?

A) Runaway loop -> grounding SLI
B) Tool misuse -> tool-call-validity SLI
C) Bad output -> loop-health SLI
D) Tool misuse -> availability SLI

### Question 12

A blameless postmortem root cause should read like:

A) "The engineer shipped a bad prompt"
B) "The model hallucinated"
C) "A prompt change reached production with no eval gate on faithfulness"
D) "The on-call was too slow"

---

## Short answer

### Scenario 1

A burn-rate alert on the loop-health SLI fires at 02:14 and the cost SLI is climbing. Walk through the first four moves a senior on-call should make, in order, and name the containment lever you'd reach for first and why.

### Scenario 2

Your postmortem's action items are all of type "prevent." Explain what's wrong with that balance and give one "detect" and one "mitigate" item for a faithfulness-regression incident.

---

## Answer key

**Q1: B.** Availability counts a fast, well-formed, wrong answer as "good." The dangerous agent failures are silent — hallucinations, tool misuse, quiet regressions — none of which throw an error (Ch. 1).

**Q2: C.** An SLI is a measured ratio of good/valid events. The target over a window is the SLO; the leftover is the error budget (Ch. 1).

**Q3: C.** Quality is judged on a sample; compute success over the judged subset and report the confidence interval. Treating unlabeled runs as success/failure biases the number; a quiet window must not read as 100% (Ch. 1).

**Q4: B.** Burn-rate alerts on how fast you consume the error budget, catching both the cliff (fast burn) and the slow slide (slow burn) — pair it with baseline-drift to catch regressions above the absolute SLO (Ch. 1).

**Q5: B.** The error budget is a release gate: a blown quality budget stops prompt/model/tool changes until reliability is restored — sharper for agents because a recent change is usually the cause (Ch. 1).

**Q6: B.** The agent chooses its own number of model/tool calls and context size at runtime, so a single request's cost is not fixed in advance (Ch. 2).

**Q7: C.** Check before each call so you never overshoot by a full expensive step; charge after so your numbers reflect real usage (Ch. 2).

**Q8: B.** A per-run cap bounds one request; it can't see a tenant flooding many individually-cheap runs. That's the per-tenant breaker's job (Ch. 2).

**Q9: B.** An open breaker rejects fast with no model spend — exactly what you want when the tenant is the source of the incident — and emits a metric (Ch. 2).

**Q10: B.** Fail closed = stop and return a graceful bounded result, never silently keep spending (Ch. 2).

**Q11: B.** Tool misuse shows up as a drop in tool-call validity. Runaway loops map to loop-health, bad outputs to grounding/success — the SLIs are the detection layer (Ch. 3).

**Q12: C.** Blameless means a system statement pointing at a missing control, not a person or "the model misbehaved." Options A, B, and D are dead ends (Ch. 4).

**Scenario 1 (model answer):** (1) Declare the incident early and take command (commander does not also type fixes; assign a scribe). (2) Triage — pull traces for the spiking runs and confirm the loop signature (e.g., two tool calls alternating). (3) Correlate — "what changed in the last 24h?" and match the timeline. (4) Contain. First lever: **roll back the correlated change**, because agent incidents correlate strongly with a recent prompt/model/tool deploy and rollback is the fastest, lowest-risk way to stop the bleeding; back it with a config tightening of the per-run step/token cap so stragglers fail closed. Then verify SLIs recover and hold open one SLI window before standing down (Ch. 3).

**Scenario 2 (model answer):** If every item is "prevent," you've done nothing to catch the next occurrence faster or shrink its blast radius — you'll re-prevent variants forever while detection and mitigation stay weak. A balanced set spans prevent/detect/mitigate. Example **detect** item: add a baseline-drift alert on the grounding SLI so a faithfulness regression pages before users report it. Example **mitigate** item: add a runtime faithfulness check that degrades low-scoring responses to human-in-the-loop review, bounding user impact when a regression slips through (Ch. 4).
