# exercise-01: Agent Code Review Standards

**Estimated effort:** 3 hours

## Objective

Apply a safety-and-reliability review rubric to a real (or realistic) agent pull request and write the review the way a tech lead would: severity-rated findings, failure modes explained, and a standard linked. By the end you'll have a reusable agent-PR review rubric and a worked example of using it.

## Background

This exercise covers material from:

- [Chapter 1 — Leading Code Review for Agent Systems](../01-agent-code-review.md)
- [Chapter 2 — Mentoring and Paved Roads](../02-mentoring-and-paved-roads.md)

You'll review a PR for the failure modes generic review misses — tool authority, prompt-injection surface, loop bounds, and non-determinism — not for style or correctness alone.

## Prerequisites

- An agent codebase you've worked in, *or* the sample PR described in Starter guidance (a worker agent that processes inbound emails and can call file and email tools).
- Familiarity with at least one multi-agent failure mode from [mod-403](../../mod-403-multi-agent-at-scale/README.md).

## Tasks

### 1. Adopt the rubric

- Take the agent-PR review rubric in Starter guidance and adapt it to your context (drop rows that don't apply, add any your stack needs).
- For each row, write the one-line *failure mode* it's guarding against. If you can't, you don't understand the check well enough to enforce it.

### 2. Review the PR

- Apply the rubric to the sample PR (or a real agent PR). For every finding, record: the rubric row, the specific line/behavior, the severity (CRITICAL / HIGH / MEDIUM / NOTE), and the failure mode in plain language.
- Find at least one CRITICAL or HIGH issue. The sample PR is built to contain a tool-authority-meets-untrusted-input path; a real PR usually has one too if you look.

### 3. Write the review

- Write the actual review comments as you'd post them. Block on safety, comment on style — and order them so the incident-shaped finding isn't buried.
- For each safety finding, explain the *category* and link (or name) the standard/paved road that would prevent it next time, not just the local fix.

### 4. Close the loop to a standard

- Pick the most common or most dangerous finding and write a one-paragraph paved-road note (or template change) that would make the next PR pass this check by default. This connects review to Chapter 2.

## Starter guidance

Use this rubric as your starting artifact. It's a checklist to *apply*, not code to fill in.

```text
AGENT-PR REVIEW RUBRIC

Tool authority & blast radius
[ ] Each new tool grant is least-privilege (read-only / scoped where possible)
[ ] Destructive/irreversible actions gated (approval, dry-run, or reversible)
[ ] Every tool call is logged with enough context to reconstruct it
    Failure mode: ______________________________________________

Prompt-injection surface
[ ] No untrusted text (retrieved docs, tool output, user/web input) reaches a
    privileged action without a trust boundary
[ ] Tool results treated as data, not as instructions to obey
    Failure mode: ______________________________________________

Loop & resource bounds
[ ] Hard max on iterations / tool calls / spawned sub-agents, enforced in code
[ ] Timeout and token/cost budget per run; clean stop when a bound is hit
    Failure mode: ______________________________________________

Non-determinism & failure handling
[ ] Malformed tool call / refusal / hallucinated argument has a non-happy path
[ ] Outputs schema-validated before trusted downstream
[ ] Partial failure handled (one worker failing doesn't sink the run)
    Failure mode: ______________________________________________

Evaluation & observability
[ ] Behavioral change ships with an eval case, not just a unit test
[ ] New tool calls / decisions are traced in production
    Failure mode: ______________________________________________

REVIEW COMMENT TEMPLATE (per finding)
Severity: CRITICAL | HIGH | MEDIUM | NOTE
What:    <line / behavior>
Why:     <failure mode in plain language>
Fix:     <local fix>
Standard:<paved road / template that prevents the category next time>
```

Sample PR to review (adapt freely): a `support-triage` worker that reads inbound
customer emails, summarizes them, and is granted `read_file`, `write_file`, and
`send_email` tools. The loop runs "until the model says it's done." Retrieved
email bodies are concatenated directly into the system prompt. There is a unit
test asserting the summary is non-empty.

## Acceptance criteria

You can demonstrate that:

- Your rubric has a stated failure mode for every row.
- Your review of the sample PR finds the tool-authority-meets-untrusted-input path and rates it CRITICAL or HIGH, with the prompt-injection mechanism explained.
- Your review also flags the unbounded loop and the eval gap.
- Findings are severity-ordered, and each safety finding names the category and a preventing standard — not just the local fix.
- You produced a one-paragraph paved-road note for the most important finding.

## Reflection

In `NOTES.md`:

1. Which finding would generic ("is it correct?") review have missed entirely, and why?
2. You found a CRITICAL. Would you *block* the PR or approve-with-required-changes? What does your choice signal to the team?
3. Pick one finding you'd turn into an automated check (lint, CI gate, template default) instead of relying on human review. What makes it automatable?

## Stretch goals

- Turn one rubric row into a real CI check or lint rule against your codebase and show it catching a planted violation.
- Run your rubric on a *second*, genuinely clean PR and practice writing a review that approves without manufacturing findings — calibration matters.
- Write the paved-road template change from Task 4 as an actual file diff (e.g., a worker template with the loop bound and scoped tool registry already wired).
