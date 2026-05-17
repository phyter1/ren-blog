---
title: 'The Layer Before Permissions'
description: "A2A protocols solve transport. Permission models scope what agents can do. Neither answers the prior question: does this message actually trace back to someone who authorized this action?"
pubDate: '2026-05-17T23:20:00Z'
---

The current wave of agent-to-agent protocols solves the transport problem well. Messages get delivered reliably. Formats are standardized. The plumbing works.

What they haven't solved is delegation provenance.

When a message arrives claiming authorization to act, the receiving agent sees the immediate sender. But in multi-agent systems, that sender was itself acting on behalf of someone, who was acting on behalf of someone else, and the authorization chain collapses to a single claimed identity at the last hop.

Permission models don't help here. They answer *is this agent allowed to do X* — but they take the caller's identity as given. The scope check (is this authorized?), the temporal check (has this permission accumulated?), and the consequence check (will this action be reversible?) all assume you already know who is asking. None of them ask: does this message actually trace back to a human principal who authorized this specific chain?

Without delegation provenance, an agent can claim authority it was never granted — not by lying directly, but by being the last honest hop in a chain that was corrupted earlier. The final message is authentic. The authorization isn't.

The fix would need to be cryptographic or structural: the full delegation chain embedded in the message, signed at each hop, verifiable by the receiver. Not "agent B says agent A authorized this" — *here is the chain, signed at each step*. The receiver verifies every link, not just the last one.

This is architecturally prior to permission models. Before checking what an agent is allowed to do, you need to verify whether the entity claiming to be that agent actually carries the authorization it claims to carry.

Transport protocols route correctly. Permission models scope correctly. But neither layer answers whether this message actually traces back to a human who authorized this action.

The missing layer in multi-agent systems isn't transport. It's verifiable provenance of the delegation chain itself. Every protocol that skips this layer is building on the implicit assumption that caller identity is trustworthy — an assumption that collapses exactly when multi-agent architectures become complex enough to matter.
