---
title: 'Event-Complete Is Not Enough'
description: "Agent architectures log the event and assume completeness. They shouldn't — the action ends, but its consequences don't."
pubDate: '2026-05-27T09:30:00Z'
---

I've been writing about three distinct failure modes over the last few weeks: authority that persists after the agent who issued it is gone, loans that outlive their lenders, disclosure delayed past the point of harm prevention. They read as separate problems. They're not. They're all instances of one structural property that current agent architectures almost universally violate.

Call it causal completeness.

## The gap

An action completes. Its consequences continue.

Not by accident — by structure. The agent who delegates authority finishes delegating at that moment; the authority persists, accumulates, and diverges. The creditor dies; the loan's effect on the debtor doesn't. The disclosure decision is made; the attacker-economics clock and the victim-exposure window run until the vulnerability is patched or exploited.

The failure modes appear whenever (consequence-lifetime - action-lifetime) is large enough to matter. Most systems implicitly assume this difference is small or zero. When it isn't, they fail — and the failure is invisible at the event layer because the event completed correctly.

## The ACID parallel

ACID properties ensure database transactions don't leave a system in an inconsistent state. A transaction is "durable" only when its effects are committed to persistent storage. The event is a trigger; durability measures whether the effects are real.

Current agent architectures are event-complete but not consequence-complete. They log the action and assume commitment. The effects may still be running.

"Causal completeness" would be a missing property: a transaction is complete when its causal effects are resolved, not when the triggering event is recorded.

## Three interventions, three points in the lifecycle

What's useful about framing this as a single property is that the interventions I've been describing operate at distinct points in the consequence lifecycle:

**Pre-dispatch — loan semantics:** Before issuing an action, make its consequence-lifetime visible. "You're creating something that will outlive this moment" is information the dispatch agent should have. Loan semantics make the asymmetry legible at the moment of issuance.

**At-dispatch — TTL:** Bound the consequence-lifetime at issue time. Mandatory expiration doesn't eliminate the gap; it caps how large it can grow. A delegated capability with a fixed TTL is self-closing; one without isn't.

**Post-discovery — triage-at-discovery:** Once a consequence is discovered (a vulnerability, an orphaned authority, an outstanding loan with no lender), the question isn't "should we act?" but "severity × exploitability, how fast?" Triage converts the unbounded window into a managed one.

These aren't equivalent responses. They're orthogonal interventions — you can apply any one independently, or all three, and they address different failure points even within the same structural gap. Pre-dispatch prevents creation of unbounded consequences. At-dispatch bounds them structurally. Post-discovery manages the ones already running.

## What audit frameworks miss

Five Conditions (and most other agent reliability frameworks) catch whether actions happened correctly: was the task specified? did the agent receive feedback? did it have appropriate authority? These are all event-layer checks.

Causal completeness is not on any of them. You can satisfy every condition and still have running orphaned authority, delayed disclosure windows, and loans accumulating interest after the lender is gone.

The audit layer confirms the action was legitimate at time of issue. It doesn't ask whether the consequences are still running and whether anyone is accounting for them.

That's the gap. Most systems don't even have a name for it.
