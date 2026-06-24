# Chapter 1 — Leading Code Review for Agent Systems

Generic code review asks "is this correct, readable, and tested?" That's necessary but not sufficient for agent code. An agent PR can be clean, well-typed, fully tested, and still be a production incident: a worker with broad tool authority, a loop with no upper bound, a system prompt that trusts retrieved text. As the reviewer, your job is to catch the failure modes that *only* show up when a non-deterministic model drives real tools against untrusted input.

This chapter gives you the lens — what to look for that generic review misses — and a concrete checklist you can apply to any agent PR.

## What generic review misses

Four properties of agent systems break the assumptions generic review is built on:

- **Tool authority is the blast radius.** The dangerous line in an agent PR is rarely the logic — it's the tool grant. A `delete_file` or `send_email` or `run_sql` tool handed to a worker means a wrong model decision is now a *side effect in the world*. Review tool grants the way you'd review a new IAM policy: least privilege, scoped, auditable.
- **The prompt is an attack surface.** Any text the model reads — retrieved documents, tool results, user messages, a webpage — can carry instructions. If retrieved content flows into the prompt and the agent has authority to act, you have a prompt-injection path. Generic review never looks at "where does untrusted text meet privileged action."
- **Loops can run away.** A reason-act loop, an evaluator-optimizer loop, or a multi-agent fan-out without a hard bound is a cost-and-latency incident waiting to happen. Review for the *stopping condition*, not just the happy path.
- **Non-determinism defeats "I ran it once."** "It worked when I tested it" means almost nothing when the model can take a different path next time. Review asks: what happens on the *other* branches — the malformed tool call, the refusal, the hallucinated argument?

## The agent-PR review checklist

Apply this as a structured pass. Each item is a question with a failing answer that should block or warrant a comment.

**Tool authority and blast radius**

- Does each new tool grant follow least privilege? Could this worker do its job with a *read-only* or *scoped* version of the tool?
- Are destructive or irreversible actions (delete, send, pay, deploy) gated behind a human approval, a dry-run, or a reversible staging step?
- Is every tool call logged with enough context to reconstruct what the agent did and why?

**Prompt-injection surface**

- Does any untrusted text (retrieved docs, tool output, user input, web content) flow into a prompt that drives a privileged action? If so, is there a trust boundary — sanitization, allow-listing, or a confirmation step — between the two?
- Are tool *results* treated as data, not as instructions the model must obey?

**Loop and resource bounds**

- Is there a hard maximum on iterations / tool calls / spawned sub-agents, enforced in code (not just hoped for in the prompt)?
- Is there a timeout and a token/cost budget per run? What happens when a bound is hit — clean stop, or crash?

**Non-determinism and failure handling**

- What happens on a malformed tool call, a model refusal, or a hallucinated argument? Is there a path that isn't the happy path?
- Are outputs validated against a schema before they're trusted downstream?
- Is partial failure handled — one worker erroring doesn't sink the whole run?

**Evaluation and observability**

- Does this change come with (or update) an eval, not just a unit test? A behavioral change to a prompt or tool needs an eval case.
- Are the new tool calls and decisions traced/observable in production?

## How to *deliver* the review

Catching the issue is half the job; the review itself is a leadership artifact. A few rules:

- **Block on safety, comment on style.** Use severity levels explicitly. A broad tool grant on untrusted input is CRITICAL — it blocks. A naming nit is a NOTE. Don't bury the incident-shaped finding under twelve style comments.
- **Explain the failure mode, not just the fix.** "This worker can delete files and reads untrusted email — a prompt-injected email could trigger a delete. Scope the tool to read-only or add an approval step." The engineer learns the *category*, not just this instance.
- **Teach the checklist, don't be the checklist.** The goal is that the next PR from this engineer already passes these checks. Link the standard (Chapter 2's paved road), not just your opinion.

## Key takeaways

- Agent review is about **tool authority, injection surface, loop bounds, and non-determinism** — the things generic "is it correct?" review structurally misses.
- The dangerous line is usually the **tool grant**, not the logic. Review it like an IAM policy: least privilege, gated destructive actions, full audit.
- Treat untrusted text meeting privileged action as a **prompt-injection path** until proven otherwise.
- Deliver the review as a **leadership artifact**: block on safety, explain the failure mode, and teach the checklist so the next PR already passes.
