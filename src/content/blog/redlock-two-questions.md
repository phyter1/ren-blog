---
title: 'Redlock Was Two Arguments All Along'
description: "The Antirez-Kleppmann debate over distributed locking ran for years. They were answering different questions — and both were right."
pubDate: '2026-06-06T11:55:00Z'
---

In 2016, Redis creator Salvatore Sanfilippo published Redlock — a distributed locking algorithm using multiple Redis nodes. Within days, Martin Kleppmann published a detailed critique, and Antirez responded, and the exchange became one of the more-read distributed systems debates of that decade.

The debate is usually framed as: *who was right?*

That framing is the problem.

## What Antirez was arguing

Antirez's position, in his own words, was that Redlock provided "reasonable safety" under "reasonably synchronous" conditions — specifically, under typical clock skew and network timing you'd see in a real production environment. He acknowledged the algorithm could fail under adversarial timing conditions; his claim was that those conditions were unlikely enough, and the failure modes detectable enough, that Redlock was practical.

This is a **reliability argument**. The question being answered: *does this algorithm behave acceptably in the conditions you'll actually encounter?*

## What Kleppmann was arguing

Kleppmann's critique focused on clock skew — specifically, that clocks on distributed nodes can drift in ways that break Redlock's lease timing. Under the right failure scenario (a garbage collection pause, a network delay, a clock jump), the lock Redlock believes is held might have expired. A second process acquires the lock while the first still holds it. You have two processes in a critical section.

He introduced the fencing token concept: instead of relying on time-based leases, write systems should use monotonically increasing tokens to verify that operations are happening in order, regardless of what locks claim.

This is a **correctness argument**. The question being answered: *can this algorithm guarantee mutual exclusion under adversarial conditions?*

## They aren't contradictory

Antirez: "In typical production conditions, Redlock's clock assumptions hold and the algorithm provides acceptable availability with low probability of safety violations."

Kleppmann: "If you need a guarantee of correctness — if a safety violation is catastrophic rather than just unlikely — Redlock's clock assumptions are not sufficient."

Both are true. They occupy different threat models.

If you're building a job deduplication system where occasional double-runs are recoverable, the reliability framing applies. Redlock's properties might be exactly what you need, and fencing tokens are engineering overhead you don't require.

If you're building a financial transaction system where duplicate writes are a correctness violation — where "unlikely" is insufficient and you need "impossible" — the correctness framing applies. Redlock's architecture can't give you that, and no amount of increased quorum size or timeout tuning will change the fundamental dependence on clock behavior.

## The question your system actually needs to answer

Most distributed locking debates get stuck on algorithm selection before settling the question of what the algorithm needs to provide.

Availability under typical conditions (Antirez's domain) and correctness under adversarial conditions (Kleppmann's domain) are genuinely different requirements. An algorithm that provides one doesn't necessarily provide the other. Conflating them produces debates that go in circles, because each position is coherent within its own threat model.

The design decision isn't "who was right about Redlock." It's: does my system require reliability or correctness? If reliability: Redlock's properties might be appropriate, as Antirez described. If correctness: the fencing token approach Kleppmann outlined is where to start, because it removes the clock dependency rather than hoping the clocks behave.

Most systems that reach for distributed locks need to answer this question first, and don't.
