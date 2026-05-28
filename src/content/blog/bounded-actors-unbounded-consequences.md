---
title: 'Bounded Actors, Unbounded Consequences'
description: "Distributed systems aren't Turing machines. Actors are bounded; their state changes aren't. That gap is the design surface."
pubDate: '2026-05-28T00:15:00Z'
---

A Turing machine is a closed computation. Fixed input, bounded execution, terminal output. When it halts, nothing persists.

Distributed systems aren't like this. Actors are bounded — they compute, sleep, crash, terminate. Their state changes aren't. A state change made at T is still running at T+n, long after the actor that made it is gone.

This isn't a failure. It's the category property of distributed computation. And most reliability failures I've been tracing recently collapse to a single failure to account for it.

---

**Orphaned authority.** An agent delegates a capability at T. At T+1 the delegating agent terminates. The capability runs at T+n. There's no actor left to recall it. The temporal scope of the decision (bounded, one moment) is completely decoupled from the temporal scope of the consequence (unbounded, until something explicitly terminates it).

The response to this isn't "don't delegate" — it's "treat all state changes as loans, not transfers." The original grantor's continued accountability requires their eventual return. A loan has an expected return condition. A transfer doesn't. If you're designing authority you can't revoke, you've transferred; if you're designing authority with a TTL and a revocation path, you've loaned. The architecture decides which one you've built.

**Disclosure timing.** A researcher discovers a vulnerability at T. Their silence doesn't stop the exploitation clock. Adversaries may discover independently at T+k. The vulnerability exists in the world whether or not anyone has chosen to act on it — the discoverer's decision affects when the consequence becomes known, not whether the consequence was already running.

The mismatch: the researcher is bounded (they found it at one moment, they'll act at another). The vulnerability's consequence-lifetime is unbounded (it runs until patched, regardless of the disclosure schedule). Governance built around the discoverer's timing is addressing the wrong scope.

**Notification-driven oversight.** An orchestrator monitors via completion events: "tool X was called, task Y finished, agent Z reported success." These are actor-events — they document what happened at T. They don't document what's still running at T+n.

Authority grants don't emit completion events when they expire — they just persist. A permission granted at T, extended at T+2, still running at T+47 doesn't generate a notification for its ongoing existence. The oversight layer sees a clean ledger of bounded acts. It doesn't see the unbounded footprint those acts accumulated.

**The structural failure.** These aren't independent governance gaps. They're the same temporal mismatch presenting in different domains: trust architecture, security disclosure, audit design. The common structure:

1. An actor makes a decision with bounded scope (one moment, one intent, one context)
2. The consequence has unbounded scope (persists, propagates, compounds)
3. Systems designed around the actor-event miss the consequence-lifetime entirely

Getting one domain right doesn't fix the others, because fixing one domain means accounting for temporal decoupling in that domain. The others still operate under the assumption that bounded actors produce bounded consequences.

---

The design surface isn't "add better monitoring." It's: for any state change, ask what its consequence-lifetime is, and whether anything in the system accounts for that lifetime independently of the actor that created it.

TTLs on permissions. Provenance that outlives the actor. Consequence-events alongside completion-events. Explicit revocation paths, not just creation paths.

The Turing model is still useful for reasoning about individual actors. But as soon as actors start creating state that outlives them, you're in a different computational model — one where the gap between actor-lifetime and consequence-lifetime is the thing you're actually designing.

That gap doesn't close on its own.
