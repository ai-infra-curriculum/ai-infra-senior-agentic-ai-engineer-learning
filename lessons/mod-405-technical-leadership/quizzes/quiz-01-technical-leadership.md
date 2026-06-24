# Module 405: Technical Leadership for Agentic Teams — Quiz

10 questions. 70% pass.

### 1. The line in an agent PR most likely to be the production blast radius is usually:

- [ ] a) A naming inconsistency in the worker class
- [x] b) The tool grant — what the agent is authorized to do in the world
- [ ] c) The choice of JSON vs. YAML for config
- [ ] d) The model temperature setting

### 2. A worker reads untrusted email bodies into its prompt and can call a `send_email` tool. This is primarily a:

- [ ] a) Latency problem
- [ ] b) Code-style problem
- [x] c) Prompt-injection path — untrusted text meets a privileged action
- [ ] d) Token-cost problem

### 3. "It worked when I ran it once" is weak evidence for an agent PR because:

- [ ] a) The reviewer didn't see the screen recording
- [x] b) Non-determinism means the model may take a different (untested) path next time
- [ ] c) Unit tests are always sufficient
- [ ] d) The CI cache was stale

### 4. A reason-act loop that runs "until the model says it's done," with no code-enforced bound, should be reviewed as:

- [ ] a) Fine, because the prompt asks the model to stop
- [x] b) A runaway-loop risk — bounds must be enforced in code, not hoped for in the prompt
- [ ] c) A style nit
- [ ] d) Acceptable as long as there's a unit test

### 5. The core idea of a paved road is to:

- [x] a) Make the safe path the *easy* path so engineers fall into the pit of success
- [ ] b) Mandate one approach and block all alternatives
- [ ] c) Replace code review with documentation
- [ ] d) Centralize all agent work under the tech lead

### 6. A standard most reliably gets adopted when it is:

- [ ] a) A 40-page policy document in the wiki
- [ ] b) Enforced only by the tech lead's manual review
- [x] c) The default, shipped with a working example and a one-line *why*
- [ ] d) Announced once in a meeting

### 7. The distinction between an architect's design doc and a tech lead's build plan is that the design owns:

- [ ] a) Build order and interface contracts
- [x] b) *What the system is and why* (components, boundaries, invariants), while the lead owns *how and in what order*
- [ ] c) The team's standup schedule
- [ ] d) Nothing — they are the same artifact

### 8. When execution reveals the design can't be built as drawn, the right move is to:

- [ ] a) Silently work around it so the team isn't blocked
- [x] b) Take the specific finding back to the architect as information for a decision
- [ ] c) Abandon the design and improvise
- [ ] d) Wait until the end of the quarter to mention it

### 9. For an agentic initiative, the riskiest assumption to test first with the spine slice is almost always:

- [ ] a) Whether the UI is visually polished
- [ ] b) Whether the database schema is normalized
- [x] c) Task reliability — does the agent do the job well/safely enough, measured against a bar
- [ ] d) Whether the logo matches brand guidelines

### 10. When de-risk-first conflicts with ship-value-early (the riskiest work isn't the most visible), the usual call is:

- [ ] a) Ship the visible value first; it builds momentum
- [x] b) De-risk first — building visible value on an unproven core just creates more to throw away — but say so explicitly
- [ ] c) Do both fully in parallel regardless of dependencies
- [ ] d) Cancel the initiative

---

## Answer key + rationale

1. **b** — Agent logic can be clean while the tool grant hands a non-deterministic model real-world authority; review tool grants like IAM policies, least-privilege and gated.
2. **c** — Any untrusted text the model reads can carry instructions; combined with a privileged tool, that's a prompt-injection path that needs a trust boundary.
3. **b** — A different model path on the next run can hit the malformed-call, refusal, or hallucinated-argument branches a single manual run never exercised.
4. **b** — Stopping conditions must be enforced in code; a prompt-level "stop when done" is not a bound and is a cost/latency incident waiting to happen.
5. **a** — A paved road bakes the safe defaults into the easy path; it's supported and optional, not a mandate or a wall.
6. **c** — Standards stick when they're the default with a working example and a brief rationale; wiki rules and one-off announcements get routed around.
7. **b** — Architects own *what and why* (invariants, trade-offs); the tech lead owns *how and in what order* because they know the team and codebase.
8. **b** — Reality that challenges the design is information, not failure; surface the specific finding to the architect so a decision is made, never a silent workaround.
9. **c** — The core bet of an agentic initiative is reliability; prove it with a thin end-to-end slice against a measurable bar before building around it.
10. **b** — Shipping visible value on an unproven core multiplies rework; de-risk first, but make the trade-off explicit so stakeholders accept a "boring" slice one.
