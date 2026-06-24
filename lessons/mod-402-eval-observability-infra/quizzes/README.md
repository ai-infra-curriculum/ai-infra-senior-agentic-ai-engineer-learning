# mod-402 Knowledge Check — Eval & Observability Infrastructure

Ten questions across the four chapters. Answers and rationale are at the bottom — try the whole set before scrolling.

## Questions

### 1. The grader interface

Why does a reusable harness require every grader — trajectory, tool-call, and LLM-judge — to conform to one `Grader` protocol returning a normalized `0.0–1.0` score?

- A. It makes the code shorter.
- B. It lets heterogeneous graders aggregate into one number and threshold uniformly.
- C. It is required by OpenTelemetry.
- D. It removes the need for datasets.

### 2. The agent-agnostic boundary

What is the boundary that makes a harness reusable across teams?

- A. The harness imports each team's agent class directly.
- B. The harness takes a callable `agent_fn(task) -> Trajectory`.
- C. Each team forks the harness and edits it.
- D. The harness only works with one model provider.

### 3. Datasets as artifacts

Why store eval datasets as versioned files rather than literals inside a test?

- A. JSONL runs faster than Python lists.
- B. So a score change is unambiguous — you know whether the agent regressed or the test set changed.
- C. Because graders can't read Python.
- D. To avoid using `pytest`.

### 4. Fail soft

In a 200-case run, case 3 raises an exception. What should a CI-ready harness do?

- A. Crash the run so the failure is visible.
- B. Record case 3 as failed and continue, still producing a report.
- C. Skip eval entirely.
- D. Retry case 3 forever.

### 5. GenAI conventions across a fleet

Why mandate the OTel GenAI semantic conventions (`gen_ai.*` keys) across every service?

- A. They make spans smaller.
- B. So backends compute cost, latency, and token charts automatically and dashboards work across services.
- C. They are only needed for one provider.
- D. They replace the need for resource attributes.

### 6. Resource attributes

What do resource attributes like `service.name` and `deployment.environment` enable that span attributes alone do not?

- A. Faster model calls.
- B. Grouping and slicing a fleet — by service, environment, and version.
- C. Cheaper tokens.
- D. Automatic redaction.

### 7. Regression threshold

Why is a regression threshold (max drop vs. the last green baseline) usually better than a fixed absolute threshold for protecting a fleet?

- A. It never fails.
- B. It adapts as the agent improves and catches deltas rather than a fixed line.
- C. It needs no dataset.
- D. It only works in production.

### 8. Keeping the gate from flaking

Which set of measures most directly keeps an LLM-eval gate stable enough that teams won't disable it?

- A. Fewer cases and a random judge.
- B. Enough cases, a pinned judge model/temperature, a tolerance band, and quarantining flaky cases.
- C. Running it only after deploy.
- D. Removing the judge grader entirely.

### 9. Where the gate runs

Where does the merge-blocking gate belong, and what runs on every push?

- A. After deploy; nothing on push.
- B. In the PR pipeline before merge; a fast smoke suite on every push.
- C. Only nightly.
- D. On the user's machine in production.

### 10. Paved road

What makes eval/observability a paved road rather than a doc?

- A. A detailed wiki page teams are told to follow.
- B. Defaults and tooling (template, versioned library, reusable CI workflow) that make the instrumented path easier than the unpaved one.
- C. A mandate from leadership with no tooling.
- D. Forbidding any escape hatches.

## Answer key

1. **B** — Normalizing to `0.0–1.0` behind one protocol is what lets you average heterogeneous graders and threshold them the same way.
2. **B** — Taking `agent_fn(task) -> Trajectory` means any team that can produce a trajectory can use the harness unchanged.
3. **B** — Versioned datasets make "the score dropped" unambiguous: agent regression vs. an edited test set, recorded in every report.
4. **B** — Fail soft: record the failed case and continue. A harness that crashes mid-run is useless in CI.
5. **B** — Standard `gen_ai.*` keys let any OTLP backend render cost/latency/tokens and make cross-service dashboards possible.
6. **B** — Resource attributes describe the emitting service, so they're how you group and partition a fleet (service, env, version).
7. **B** — A regression threshold adapts as the agent improves and catches deltas vs. the last green baseline, which is what protects a fleet.
8. **B** — Enough cases, a pinned judge, a tolerance band sized from measured variance, and quarantining flaky cases keep the gate stable.
9. **B** — The full, judge-backed gate blocks merges in the PR pipeline; a fast smoke suite runs on every push for quick feedback.
10. **B** — A paved road is defaults and tooling that make the instrumented path the easy one — adoption by default, with escape hatches preserved.
