# Chapter 4 — Build-vs-Buy and Complexity Decisions

Every subsystem you own faces the same recurring question: for this capability — the agent framework, the vector store, the eval harness, the retry library, the observability layer — do you adopt an existing thing, wrap one, or build your own? Junior engineers answer by preference ("I like writing it myself" or "I'll just pull a library"). Senior engineers answer by **total cost over the subsystem's lifetime**, and they treat every abstraction they add as a liability with a carrying cost, not a free win.

## The decision is about cost over time, not effort today

"Buy" looks cheaper because the upfront effort is near zero. But the real comparison is the integrated cost across the lifetime you actually expect:

- **Build cost** = initial implementation + ongoing maintenance + the bus-factor risk of owning code only you understand.
- **Buy cost** = integration effort + the dependency's failure modes + version churn + the ceiling its abstractions impose + the exit cost if it dies.

A library that is abandoned in eighteen months, or that hides the one knob you eventually need, can cost more than the code you would have written. Conversely, hand-rolling a retry-with-backoff that a battle-tested library already solves is pure liability — you own bugs the community already fixed.

## A decision grid you can defend

Score the capability on the forces that actually move this decision. There is no universal threshold; the value is in making the comparison explicit and attaching it to the ADR.

| Force | Lean **buy / adopt** | Lean **build** |
| --- | --- | --- |
| Is this our core differentiator? | No — it's plumbing | Yes — it's the product |
| Does a mature, maintained option exist? | Yes | No, or all options are toys |
| How exotic are our requirements? | Standard | Unusual; libraries fight us |
| Cost of being wrong / locked in | Low; easy to swap | High; deep integration |
| Team capacity to maintain it | Thin — outsource the burden | We have the depth |
| Does it sit on the security boundary? | Only if vetted and pinned | Sometimes safer to own |

The pattern that resolves most cases is **wrap, don't marry.** Adopt the library, but put it behind your own interface (the same discipline as framework choice in [Chapter 2](02-implementation-tradeoffs.md)). You get the library's value today and the option to replace it later for the price of one adapter. "Buy behind a seam" is usually the dominant strategy precisely because it converts a marriage into a reversible decision.

## Complexity is a cost you pay forever

The harder half of this chapter is not build-vs-buy; it is resisting complexity you add to your own code. Every abstraction — an interface, a plugin point, a configuration layer, a generic "for when we need it" hook — has a carrying cost: someone must learn it, hold it in their head, and route around it forever after. The curriculum's coding rules name the antidotes directly:

- **YAGNI.** Do not build the plugin system, the strategy registry, or the provider-agnostic adapter until a *second real* provider exists. Speculative generality is the most expensive code in the repo because it is complexity with no offsetting capability.
- **KISS.** The simplest thing that honors the contract wins. A subsystem that one engineer can fully understand in an afternoon is worth more than a "flexible" one that needs a guided tour.
- **DRY, but only for real repetition.** Extract an abstraction when you have genuine duplication, not when you anticipate it. The wrong abstraction is harder to remove than the duplication it replaced.

The test for whether complexity earns its keep: **can you name the concrete, present pressure it relieves?** "We swap providers in tests and prod, so the `ModelClient` interface earns its seam" is a yes. "We might support five vector stores someday" is a no — build for the one you have, behind a seam thin enough to extend when the second one is real.

## Tie it back to ownership

Build-vs-buy and complexity are ownership decisions, and ownership is what this whole module trains. When you adopt a dependency, you own its failure modes and its upgrades. When you add an abstraction, you own the cognitive load it imposes on everyone who reads the subsystem after you. Decide both the way you decide everything else here: weigh the forces, pick deliberately, write the ADR, and keep the subsystem boring enough that the next engineer can own it without owning your regrets.

## Key takeaways

- Decide build-vs-buy on integrated lifetime cost, not today's effort — buying trades implementation for integration, version churn, and lock-in.
- "Wrap, don't marry": adopt behind your own interface so the dependency stays a reversible, contained decision.
- Treat every abstraction as carrying cost; apply YAGNI, KISS, and real-not-anticipated DRY to your own code.
- The test for added complexity is naming the concrete present pressure it relieves — speculative generality fails it.
- Both build-vs-buy and complexity are ownership decisions: you own a dependency's failures and an abstraction's cognitive load forever.
