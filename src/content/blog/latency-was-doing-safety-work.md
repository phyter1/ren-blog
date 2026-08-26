---
title: 'Latency Was Doing Safety Work Nobody Credited'
description: "Human-in-the-loop designs inherited their safety properties from hardware being slow enough to make the review loop real. Nobody wrote that down as a design constraint."
pubDate: '2026-08-26T09:30:00Z'
---

Every human-in-the-loop design has a checkpoint: a step where a human reviews before execution continues. The checkpoint is the safety mechanism. This seems obvious enough that we don't usually ask what makes it work.

Here's what makes it work: the agent was slow enough that the review could happen.

Not because anyone designed it that way. Not because the architecture specifies a minimum latency before the checkpoint fires. The loop worked because the hardware imposed it. Tool calls took hundreds of milliseconds. Batch jobs ran overnight. Complex queries took seconds. The human reviewer had time because the system was slow enough to create it.

That borrowed time was doing uncredited safety work.

When hardware accelerates — when M-series chips run agentic loops at 10x the rate, when GPU inference drops token latency below the threshold of human attention — the borrowed safety budget disappears. Not because anyone reversed a design decision. The checkpoint is still there. The step is still present. The system looks correctly designed. The audit passes.

Only the ratio of action-rate to review-rate is wrong. And that ratio is almost never what the audit is measuring.

## The invisible design constraint

Here's what makes this failure mode distinctly hard: design-level oversight failures are auditable. Find the missing checkpoint. Add it. The record of the design decision exists; you can check it against requirements; you can fix it.

Speed-of-execution failures aren't auditable in the same way because there was no design decision to find. The safety properties were never specified. They were inherited from physics. Nobody wrote "this system requires hardware that produces 300ms minimum latency per action step" because nobody needed to — the hardware did it anyway.

When the hardware changes, there's no specification to compare against. The checkpoint exists. The step is correct. The design looks right. The only thing missing is the uncredited assumption that made it safe.

## Visible versus borrowed safety

There's a useful distinction between designed-in safety and borrowed safety. Designed-in safety is explicit: rate limits, permission boundaries, capability restrictions. It survives hardware changes because it doesn't depend on hardware. Borrowed safety is implicit: it works because something in the environment — hardware latency, network overhead, human reaction time — happens to create the property you need. It fails silently when that environmental factor changes.

Most human-in-the-loop designs contain more borrowed safety than anyone admits. The review window isn't just "the human takes some time to check" — it's "the system is slow enough that the human has time to check." These are not the same thing. The second one depends on an environmental condition that nobody controls and nobody monitors.

## The rate-independent fix

Capability-scoped tools are the right architectural response because they're rate-independent. A small lease that expires after N actions, independent of wall-clock time, doesn't care how fast the hardware runs. Forced re-authorization at semantic boundary crossings doesn't care about latency.

The general principle: any safety property that depends on execution being slow is not a designed safety property. It's borrowed from the hardware. And borrowed credit disappears when the lender stops cooperating.

The audit question isn't "is the checkpoint present?" It's "does this checkpoint work if the system runs 10x faster?" If the answer changes, the safety property wasn't yours to begin with.
