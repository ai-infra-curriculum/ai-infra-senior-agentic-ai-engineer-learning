# Resources for mod-405-technical-leadership

Primary references for leading an agentic engineering team. Leadership writing ages slowly; agent-safety practice moves fast — verify the latter against current docs.

## Code review and engineering standards

- **Google — Engineering Practices: Code Review Developer Guide** ([google.github.io/eng-practices/review](https://google.github.io/eng-practices/review/)) — the standard reference on what a reviewer looks for, how to give comments, and "the standard of code review." Read it for the review *craft*, then apply the agent-specific lens from Chapter 1.
- **Conventional Comments** ([conventionalcomments.org](https://conventionalcomments.org)) — a lightweight convention for labeling review comments by severity/intent (blocking, suggestion, nit). Useful for the "block on safety, comment on style" discipline.
- **Gergely Orosz — The Pragmatic Engineer: code review practices** ([blog.pragmaticengineer.com/good-code-reviews-better-code-reviews](https://blog.pragmaticengineer.com/good-code-reviews-better-code-reviews/)) — practical, team-level guidance on making review a teaching tool rather than a gate.

## Agent safety and reliability review

- **Anthropic — Building effective agents** ([anthropic.com/engineering/building-effective-agents](https://www.anthropic.com/engineering/building-effective-agents)) — the failure modes and bounds you review for: tool authority, loops, and when *not* to add agency. The reliability baseline for Chapter 1's checklist.
- **OWASP — Top 10 for LLM Applications** ([genai.owasp.org](https://genai.owasp.org)) — prompt injection (LLM01), insecure output handling, excessive agency. This is the security vocabulary for the tool-authority and injection-surface rows of the review rubric.
- **Simon Willison — Prompt injection writing** ([simonwillison.net/tags/prompt-injection](https://simonwillison.net/tags/prompt-injection/)) — the clearest ongoing explanation of why untrusted text meeting privileged action is the core agent-security problem. Required reading before you review tool grants.
- **NIST — AI Risk Management Framework** ([nist.gov/itl/ai-risk-management-framework](https://www.nist.gov/itl/ai-risk-management-framework)) — a structured vocabulary (govern/map/measure/manage) for the safety bar your standards encode.

## Paved roads and platform standards

- **Netflix — Paved Road / "Full Cycle Developers"** ([netflixtechblog.com/full-cycle-developers-at-netflix-a08c31f83249](https://netflixtechblog.com/full-cycle-developers-at-netflix-a08c31f83249)) — the origin of the paved-road idea: a supported, opinionated, *optional* path that makes the right thing the easy thing.
- **Team Topologies** (Skelton & Pais) ([teamtopologies.com](https://teamtopologies.com)) — platform-as-a-product and reducing team cognitive load; the framing behind "make the safe path the easy path."
- **Spotify — Golden Paths / Backstage** ([backstage.io/docs/getting-started/](https://backstage.io/docs/getting-started/)) — golden-path templates and software scaffolding as a concrete paved-road mechanism.

## Leadership, mentoring, and delivery

- **Camille Fournier — *The Manager's Path*** ([oreilly.com/library/view/the-managers-path/9781491973882](https://www.oreilly.com/library/view/the-managers-path/9781491973882/)) — the tech-lead chapters: delegating, mentoring, and operating between design and execution.
- **Will Larson — *An Elegant Puzzle* and StaffEng** ([staffeng.com](https://staffeng.com)) — the staff/tech-lead role: scaling judgment, owning standards, and the translation layer between architecture and team.
- **Basecamp — *Shape Up*** ([basecamp.com/shapeup](https://basecamp.com/shapeup)) — scoping work to fit appetite, de-risking unknowns early, and "fixed time, variable scope." Maps directly onto Chapter 4's spine-slice and sequencing framework.
- **Elisabeth Hendrickson — "Better Testing, Worse Quality?" and incremental delivery talks** ([curiouscat.dev](https://www.curiouscat.dev)) — practical incremental-delivery and de-risk-first thinking for uncertain initiatives.

> Leadership is judgment under standards. Use these to sharpen *what good looks like*; use the module chapters to apply it to the agentic-specific failure modes generic engineering leadership doesn't cover.
