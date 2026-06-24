# Chapter 4 — Making Eval and Observability a Paved Road

You can build the best harness ([Chapter 1](01-reusable-eval-harness.md)), the cleanest fleet tracing ([Chapter 2](02-tracing-and-dashboards.md)), and the strictest gates ([Chapter 3](03-quality-signals-and-gates.md)), and still fail — if nobody uses them. Infrastructure that requires a wiki page, a kickoff meeting, and three days of plumbing to adopt loses to the engineer who skips it. A **paved road** is the practice of making the well-instrumented, well-gated path *easier than the unpaved one*, so teams adopt it by default rather than by mandate. This is the difference between a senior engineer who builds tools and one who changes how an org works.

```text
   unpaved: copy 4 repos, read 3 docs, wire OTel by hand, write eval from scratch
   paved:   `agentkit init my-agent`  ─▶  tracing + harness + gate, working, in minutes
```

## The paved road is a default, not a document

A document describing how to instrument your agent will be skimmed and ignored. The paved road is the *default behavior of the tooling*: the project template already exports OTel with the right resource attributes, the CI workflow already runs the gate, the harness is already a dependency with a starter dataset. Adoption happens because doing nothing gives you the instrumented path, and opting out takes effort.

- **A project template / generator.** `cookiecutter` or an internal `init` command that scaffolds a new agent with tracing, a harness wiring, and a gating CI job already present.
- **A shared library, versioned.** Tracing wrapper and harness ship as a package teams pin and upgrade — not files they copy and let rot.
- **A CI template.** A reusable workflow (GitHub Actions reusable workflow, or equivalent) teams reference in one line to get the smoke + full + nightly gate structure.

## Sensible defaults over configuration

A paved road has opinions. Ship defaults that are right for 80% of agents so a new team configures *nothing* to get value: the GenAI conventions pre-wired, a default sampling and redaction policy, a starter eval suite with the four standard signals and reasonable thresholds, dashboards provisioned from a template. Let teams override when they have a real reason — but never make them assemble the baseline themselves. Configuration is a tax; defaults are the road surface.

## Golden paths and escape hatches

Document the **golden path** — the supported, paved way to do the common thing — clearly and in one place. Equally important, provide **escape hatches**: the team with a genuinely unusual agent can drop to the raw harness API or hand-instrument a span, without forking the platform. A paved road that forbids leaving it becomes a cage, and engineers route around cages. Support the common case by default; permit the uncommon case explicitly.

## Lower the activation energy relentlessly

Every step between "I have an agent" and "it's traced and gated" is a place teams fall off. Treat reducing that step count as the core work:

- **One command to onboard.** If adopting tracing is more than adding a dependency and an init call, it's too much.
- **Local-first.** A team must be able to run the harness and see traces on their laptop before touching production infra — Arize Phoenix runs locally for exactly this.
- **Good errors.** When a team misconfigures the gate, the failure tells them how to fix it. Bad paved-road errors generate support tickets that erode trust in the road.
- **Migration tooling, not migration memos.** Moving teams onto the standard means a codemod or script, not a request that they each do the work.

## Measure adoption, because a paved road is a product

You built a product; measure whether it's used. The metrics that tell you the truth:

- **Coverage** — fraction of agent services emitting standard traces and running the gate. The headline number.
- **Time-to-first-trace** — how long from a new project to a trace in the dashboard. If it's climbing, the road has potholes.
- **Gate adoption** — fraction of agent repos with the merge-blocking gate enabled (and not bypassed).
- **Escape-hatch rate** — how often teams leave the golden path. A spike means a real use case the defaults don't serve — a backlog item, not a failure.

When coverage stalls, the answer is rarely "send another reminder." It's "find the friction and remove it." That instinct — own the adoption, not just the artifact — is what makes eval and observability *infrastructure* rather than a script you wrote once.

## Key takeaways

- A **paved road** wins by making the instrumented, gated path *easier* than the unpaved one — adoption by default, not by mandate.
- Ship the road as **defaults and tooling**: a project template, a versioned shared library, and a reusable CI workflow — never a doc that asks teams to wire it themselves.
- Provide a clear **golden path** *and* **escape hatches**; forbidding the uncommon case turns a road into a cage engineers route around.
- **Measure adoption** (coverage, time-to-first-trace, gate adoption, escape-hatch rate) and treat stalled coverage as friction to remove, not a reminder to send.
