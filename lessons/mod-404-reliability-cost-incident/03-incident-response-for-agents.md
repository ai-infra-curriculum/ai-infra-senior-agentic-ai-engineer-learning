# Chapter 3 — Incident Response for Agent Failures

Incident response is a well-established discipline — the SRE workbook lays out roles (incident commander, ops lead, communications lead), the detect → triage → contain → eradicate → recover loop, and the rule that **declaring an incident early is cheap and de-escalating is easy**. None of that changes for agents. What changes is the *taxonomy of failure* and the *containment levers*. This chapter maps the standard IR loop onto the three failure classes unique to agentic systems.

## The three agent-specific failure classes

- **Runaway loops.** The agent fails to converge — it re-plans, re-tries, or oscillates between two tool calls, consuming steps, tokens, and money without producing a result. Symptom: a rising tail in steps/tokens-per-run and a spike in the cost SLI.
- **Tool misuse.** The agent calls a real tool with wrong or dangerous arguments — deletes the wrong record, emails the wrong customer, queries production with an unbounded scan. Symptom: a drop in the tool-call-validity SLI, or side-effects reported downstream. This is the highest-severity class because it mutates the world.
- **Bad outputs.** The agent returns fast, well-formed, **wrong** answers — hallucinated facts, fabricated citations, off-policy advice. Symptom: a drop in grounding/success SLIs, or a wave of user reports. Hardest to detect because nothing errors.

Note the detection column maps directly onto the SLIs from Chapter 1 — that is the point. Your SLIs are not just for reporting; they are the **detection layer** of incident response.

## Detection and declaring

Detection comes from three sources, in increasing badness: an SLI burn-rate/drift alert (you caught it), a cost-SLI spike (the bill caught it), or a user/downstream report (the customer caught it). Whichever fires, the senior move is the same: **declare early**. An agent incident has a meter running — every minute of a runaway loop or a tool-misuse bug is money and, for tool misuse, irreversible side-effects. The cost of declaring and standing down is a few minutes; the cost of waiting is unbounded.

When you declare, assign the standard roles even on a small team: an **incident commander** who owns the decision-making and isn't also typing fixes, and a **scribe** who keeps a timestamped log (you will need it for the postmortem in Chapter 4). For a customer-visible incident, name a **comms** owner.

## Containment levers (agent-specific)

Containment is "stop the bleeding," and the levers differ by failure class. The senior skill is having these built **before** the incident so containment is a config flip, not a code deploy.

- **Kill switch / feature flag.** Disable the offending agent or tool path entirely. For tool misuse this is the first move — pull the tool's permission so no further side-effects occur, even before you understand the bug.
- **Tighten the budget.** For runaway loops, drop the per-run step/token cap (Chapter 2) so in-flight and new runs fail closed fast instead of looping. Because limits are config, this is immediate.
- **Trip the breaker / rate-limit.** For a blast that's tenant- or feature-scoped, open the circuit breaker for that scope to halt the spend while you investigate.
- **Roll back the change.** Agent incidents correlate strongly with a recent prompt, model, or tool change. "What deployed in the last 24h?" is the first triage question; rolling back is often the fastest containment.
- **Degrade gracefully.** Where you can't fully disable, fall back to a safer mode — a smaller model, a canned safe response, or human-in-the-loop review for the affected path.

## A worked example: the runaway loop

A burn-rate alert fires on the loop-health SLI at 02:14; the cost SLI is climbing in parallel. The on-call declares an incident and becomes commander.

1. **Triage (02:16).** Pull recent traces for the spiking runs. They show the same two tool calls alternating ~30 times before hitting the per-run step cap. The cap is *working* — runs aren't infinite — but the volume of capped runs is burning budget and returning failures to users.
2. **Correlate (02:19).** "What changed?" A retrieval-tool prompt shipped at 01:50 that returns ambiguous results, causing the planner to oscillate. The timeline matches.
3. **Contain (02:22).** Roll back the 01:50 change (fastest lever). As a belt-and-suspenders move, drop the per-run step cap from 12 to 6 via config so any stragglers fail closed sooner.
4. **Verify (02:30).** Loop-health SLI recovers toward baseline; cost SLI flattens. Hold the incident open through one full SLI window to confirm, then stand down.
5. **Capture.** The scribe's log — alert time, trace evidence, the 01:50 correlation, the rollback — becomes the spine of the postmortem.

Notice what made this fast: the SLIs *detected* it, the per-run cap *bounded* it (so it was a budget incident, not an outage), and config-driven containment *stopped* it without a deploy. That is the system you are building toward.

## Key takeaways

- The IR loop (detect → triage → contain → recover, with a commander and a scribe) is unchanged; the **failure taxonomy** is what's agent-specific.
- The three classes — **runaway loops, tool misuse, bad outputs** — each map to a Chapter 1 SLI, so your SLIs *are* the detection layer.
- **Declare early:** an agent incident has a running meter, and tool misuse causes irreversible side-effects — waiting is the expensive option.
- Build containment levers **in advance** (kill switch, config-driven budgets, breaker, rollback) so stopping the bleeding is a flip, not a deploy.
