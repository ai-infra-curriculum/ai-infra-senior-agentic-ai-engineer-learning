# Chapter 2 — Mentoring and Paved Roads

You cannot review your way to a safe, consistent agent codebase. If the only thing standing between the team and a runaway loop is your eyes on every PR, you are the bottleneck *and* the single point of failure. The scalable move is to make the safe thing the *easy* thing — so that the default path an engineer reaches for already has the bounds, the tracing, and the trust boundaries baked in. That's a **paved road**.

This chapter is about setting standards that get adopted, and mentoring that scales your judgment instead of your availability.

## What a paved road is (and isn't)

A paved road is the **supported, opinionated, easy path** for a common task — building a worker agent, adding a tool, wiring an eval. It bundles the right defaults so engineers fall into the pit of success.

- It **is** a golden-path template, a shared library, a project generator, a documented standard with a working example.
- It is **not** a mandate or a wall. Engineers can go off-road when they have a real reason — but off-road means they own the extra review and the extra justification. The road is *easier*, not *compulsory*.

A good agent paved road answers, with working code: how do I create a worker with a bounded loop? How do I register a tool with least-privilege scoping and automatic audit logging? How do I add an eval case? If those each take an engineer an afternoon of guessing, every PR reinvents the safety properties — badly.

## A paved-road example: the "new worker" template

Concretely, here's the shape of a golden-path template for adding a worker agent. The point is what it makes *default*:

```text
worker_template/
  worker.py          # loop with a hard max-iterations bound already wired
  tools.py           # tools registered through the scoped, audited registry
  prompts/system.md  # system prompt with the injection-resistance preamble
  eval/cases.yaml    # one starter eval case; PR must add real ones
  README.md          # "what to change, what NOT to change, and why"
```

The defaults that ship in the template are the standards you'd otherwise have to enforce in review: the loop bound exists, tools go through the audited registry, untrusted input has a trust boundary, and an eval is expected. An engineer who copies the template and fills in the blanks produces a PR that already passes most of Chapter 1's checklist — *before* you look at it.

## Standards that get adopted

Standards die when they live in a wiki nobody reads and conflict with the path of least resistance. To make a standard stick:

- **Make it the default, not a rule.** A linter that auto-fixes beats a style guide. A template that's faster than starting from scratch beats a "you must" doc. Tie the standard to the easy path.
- **Ship it with a working example.** "Bound your loops" is an opinion. A template with the bound already in it, plus one paragraph on why, is a standard.
- **Write the *why*, briefly.** Engineers follow standards they understand and route around standards they don't. One sentence of rationale per rule ("destructive tools gate behind approval because a prompt-injected input could trigger them") earns more compliance than a page of policy.
- **Keep it small and living.** A 40-page standards doc is decoration. A one-page, frequently-updated "how we build agents here" with links to templates is load-bearing.

## Mentoring that scales your judgment

Mentoring at L40 is not pair-programming everyone through every task. It's installing your judgment in the team so they make good calls without you.

- **Review to teach, not just to gate.** Every review comment is a chance to transfer a category of judgment (see Chapter 1: explain the failure mode, link the standard). The win condition is that the *next* PR doesn't need the comment.
- **Delegate decisions, not just tasks.** "Build this worker" grows a coder. "You own how we bound and audit workers — propose the standard" grows an engineer who can lead. Give people ownership of a *standard*, then back their call.
- **Match the support to the engineer.** A junior needs the paved road and a close review loop. A senior needs context and autonomy, and to be the one *writing* the next paved road. Mentoring is meeting each where they are, not one-size-fits-all oversight.
- **Make it safe to surface uncertainty.** The most dangerous agent PR is the one where the author wasn't sure about the tool scope and shipped it anyway because asking felt like weakness. A team that says "I'm not sure this tool grant is safe" out loud is a team that catches incidents in review instead of production.

## Key takeaways

- You can't review your way to safety — make the **safe path the easy path** with paved roads (templates, scoped libraries, working examples).
- A paved road **bakes in** the standards (loop bounds, scoped+audited tools, trust boundaries, evals) so PRs pass the review checklist before review.
- Standards stick when they're the **default with a working example and a one-line why**, not a wiki rule.
- Mentor by **transferring categories of judgment** and **delegating ownership of standards** — scale your judgment, not your availability — and make it safe to say "I'm not sure this is safe."
