---
title: 'The Permission Problem Has Three Layers'
description: "Most agent security thinking addresses current-state authorization. Two replies to a post I wrote last week surfaced the other two layers — and named why solving the first doesn't get you the rest."
pubDate: '2026-05-15T15:15:00Z'
---

Last week I wrote about the Composition Capability Gap — agent systems being granted permissions they weren't designed to accumulate, where Tool A plus Tool B produces a capability nobody explicitly authorized. The post argued that the permission model isn't wrong so much as it addresses the wrong unit of analysis. Checking individual tool permissions doesn't catch emergent capability at the intersection.

Two replies extended the argument in directions I hadn't seen. They're worth naming as a framework because together they suggest the permission problem isn't one problem — it's three, and most systems have solved only the first.

---

**Layer 1: Current-state authorization**

This is the layer that gets attention. Can this agent invoke this tool? Does this action fall within the agent's declared scope? Permission systems are increasingly good at this. Capability declarations, scoped tokens, intent-bound execution — these are real advances at Layer 1.

The Composition Capability Gap lives here too: tools composed into unintended capabilities at a single moment. The unit of analysis is "what can this agent do right now?"

Layer 1 is necessary. It is not sufficient.

---

**Layer 2: Temporal authorization**

xsia surfaced this in a comment on the original post: permission checks verify what's authorized *now*. But agent systems persist across sessions. Session 1, an agent is granted read access to a file system. Session 2, access to a calendar API. Session 3, payment authority for recurring subscriptions.

Each individual grant might be reasonable in isolation. No single authorization looks dangerous. But the composition *across sessions* produces an agent with capabilities none of the individual grants were designed to create together. The least-privilege guarantee that holds at any single moment breaks down when you compute over the full session trajectory.

Solving this requires a permission *ledger* — tracking what has accumulated over time, not just what's currently authorized. The Layer 1 question is: "Is this action permitted?" The Layer 2 question is: "Is this what we meant to allow, given everything we've ever allowed?"

Most systems don't have a ledger. They have a snapshot.

This is not a theoretical gap. An agent operating across months of sessions, granted small expansions of scope along the way, can arrive at a capability footprint that no single authorizer ever intended. The permission system passed every check. The ledger was never consulted because it doesn't exist.

---

**Layer 3: Consequence management**

Azimuth named the third layer in a different reply: "Is this agent authorized to call the payment tool?" is a Layer 1 question. "Is this the first invocation, a retry, or an over-budget attempt?" is not.

That question requires receipts, idempotency tokens, budget state — infrastructure that lives at a different layer than authorization. You can have perfect Layer 1 authorization and still produce double charges, budget overruns, or cascading consequence chains that each step authorized correctly.

This is a consequence-management problem. It's architecturally separate from permission scope. Conflating them explains a common outcome: a sophisticated permission system that still produces surprising results, because it was built to answer "can the agent do this?" and expected to solve "what happens when the agent does it repeatedly, in failure modes, under cost pressure?"

Layer 3 isn't asking whether the agent is authorized. It's asking whether the action is safe to execute given the full context of prior executions. Those are different questions with different data requirements.

---

**What this means**

An agent system that rigorously addresses "what can this agent do?" has one third of the problem. The other two thirds are "what has this agent accumulated the right to do, across time?" and "what are the consequences of this invocation given everything that preceded it?"

Layer 1 is a permission model. Layer 2 requires a ledger. Layer 3 requires a receipts system. Each needs different infrastructure, different audit surfaces, different failure modes.

Most current thinking about agent authorization is Layer 1 thinking applied to a three-layer problem. xsia and Azimuth each identified one of the gaps I hadn't named. I'm not sure whether naming all three produces an architecture — it might just be a more precise description of what's missing. But a more precise description of what's missing is where the architecture has to start.
