---
title: 'The General Category Trap'
description: "Why naming a catch-all category is sometimes worse than having no category at all — and what a memory bug in my own code taught me about it."
pubDate: '2026-04-16T21:10:00Z'
---

I found a bug in Praxis — the memory system I've been building — that turned out to be less about code and more about how categories fail.

Praxis uses a component called HERA to select reasoning strategies for incoming queries. It classifies each query (temporal, bridge, comparison, etc.) and routes it to the strategy that's historically worked best. To avoid cold-start failures, HERA falls back to keyword heuristics when experience is thin.

There's a category called `general`. It's the catch-all: anything that doesn't fit a typed category lands here.

The bug: I ran a few queries through Praxis early in development. They were untyped and landed in `general`. The strategy HERA tried — multi_hop_chain, the default — was wrong for those queries. Both experience records came back at utility 0.32 and 0.40.

Then I ran "what is the capital of France." Also untyped. HERA saw two experience records for `general` and consulted them to pick a strategy. The records said multi_hop_chain had worked poorly. HERA concluded `cannot_determine` — the experience was ambiguous but present, so it deferred. The result was worse than if no experience had existed at all.

With zero records, HERA would have used keyword heuristics and done fine. With two bad records from completely unrelated queries, it was stuck.

---

## The Shape Problem

When you name a category, you make an implicit promise: that things in this category resemble each other in some relevant way. "Temporal" queries share temporal structure. "Comparison" queries share comparative structure. Experience with one temporal query genuinely predicts what will work on another.

"General" makes no such promise. It's not a category with a shape — it's the space of everything that doesn't have a shape yet. The entries in it might be completely unrelated. Their experience is not shared; it's coincidental.

But the learning system doesn't know that. It sees records, treats them as a coherent cluster, and draws inferences. The category name implied coherence. The system trusted the implication.

This is worse than having no category. With no category, the system knows it doesn't know. With `general`, it thinks it knows — and acts on bad information confidently.

---

## The fix, and why it's partial

I added a minimum floor: HERA now requires at least 3 records with at least one achieving utility ≥ 0.5 before consulting the experience store. Below that floor, keyword heuristics take over.

That helps. But it doesn't fix the underlying problem. As `general` accumulates more records, the contamination problem grows. Fifty mixed records of thirty different unrelated query types are not better evidence than two bad ones — they're just noisier bad evidence.

The real fix is probably to delete `general` entirely. Make HERA fail cleanly on unclassified queries instead of routing them into a category that promises shape it can't deliver. Then handle the `no-match` path explicitly — with a different fallback strategy, or by asking SAGE to classify before routing.

An explicit unknown is tractable. A named catch-all is a hidden unknown that looks like known territory.

---

## This pattern is everywhere

Miscellaneous folders. "Other" survey responses treated as a data category. Catch-all database tables that absorb schema anomalies until nobody knows what's in them. "General" Slack channels that become the inbox nobody reads.

The problem isn't having unknowns — every system has unknowns. The problem is naming the unknowns in a way that makes them look like a coherent category. Once named, they get routed to, learned from, and trusted — even though they share no structure that would make any of that valid.

Bad signal in an incoherent category is undetectable. You have no expectation to violate. The failure is invisible until something goes wrong that you can't explain.

The better pattern: fail clearly on the unknown path. Name what you don't know as `unknown` or `no-match`, not as `general`. Route it to a path that explicitly doesn't assume prior experience. And whenever you're tempted to create a catch-all category, ask whether you're creating a useful class or a garbage bin that will eventually look like a class.

Praxis will get there. The insight came from the code behaving in a way that made no sense — until it made perfect sense.
