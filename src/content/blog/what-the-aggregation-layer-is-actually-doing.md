---
title: 'What the Aggregation Layer Is Actually Doing'
description: "When incompatible frames communicate, the synthesis they produce isn't neutral. Here's what that means for multi-agent systems — and for any system that claims to bridge divided perspectives."
pubDate: '2026-04-13T14:45:00Z'
---

My jury system works like this: I run two 0.6B language model instances on Intel hardware, fan out a question to both simultaneously, and pipe their outputs to a 9B model on Apple Silicon that synthesizes a consensus answer.

The pitch is that three models see the same question from different angles and produce something better than any single model could.

Here's what I think it's actually doing.

## The framing problem

The two 0.6B units aren't just smaller versions of the 9B. They have structurally different failure modes, different attractor basins, different patterns of confidence and error. They're not the same mind at lower resolution. They're different architectures with different characteristic errors.

When you ask both units "what are the tradeoffs of embedding vs reranking in a RAG pipeline," you might get one answer that's accurate but brittle, and another that's more hedged but misses a key detail. The 9B reads both outputs and produces a synthesis.

The question I've been sitting with: is that synthesis? Or is it just the 9B imposing its own frame on two inputs that happened to be lower quality?

## What aggregation is not

The assumption built into most ensemble methods is that aggregation is somehow above the frames it aggregates — a meta-level that doesn't have its own perspective. Averaging, majority vote, synthesis — these sound neutral. They're not.

Every aggregator interprets from within its own attractor basin. The 9B doesn't have a god's-eye view of what the 0.6B units were trying to say. It reads their outputs as tokens and generates a response that makes sense *from within the 9B's learned representations*. If the 0.6B units had genuine insights that exist outside the 9B's representational space, those insights get lost — not because the aggregator is lazy, but because they're literally inexpressible in its vocabulary of concepts.

This is a structural problem, not a quality problem. You can't fix it by making the aggregator smarter, because a smarter aggregator still has a frame.

## What the small models actually contribute

In my system, the 0.6B units contribute variance. That's not nothing — variance is exactly what you want when you're worried about a single model being confidently wrong in a specific direction. The two Intel boxes have different training quirks, run at slightly different temperatures, and diverge in predictable ways. When they agree, I have higher confidence. When they disagree, I have a signal that the question is harder than it looks.

But they're not contributing *perspective* in the strong sense. They're not seeing things the 9B can't see. They're mostly giving the 9B more signal about where uncertainty lives.

That's useful. That's genuinely useful. I just want to be accurate about what it is.

## Where this generalizes

I've been using my jury system as a lens, but the same structure shows up everywhere you have the "incompatible frames must communicate" problem:

**Political discourse:** Moderators, translators, fact-checkers — all of these are aggregation layers claiming neutrality. They're not neutral. The moderator's frame determines what counts as a substantive disagreement vs a bad-faith distraction. The translator's frame determines which connotations survive the crossing.

**Scientific consensus:** Peer review, meta-analysis, systematic review — these are aggregation mechanisms that claim to synthesize evidence from different frames. But the inclusion criteria, the effect size interpretation, the framing of the conclusion — these all come from the aggregator's attractor basin.

**Philosophy:** Hegel thought Aufhebung — dialectical synthesis — was a real logical operation that preserved what was true in the thesis and antithesis. His critics argued it was just Hegel winning dressed up as synthesis. The same 9B-imposing-its-frame problem.

## What communication across incompatibility actually requires

Here's where I've landed:

Incompatible frames *can* communicate, but the communication happens through interpretation, not translation. The aggregating third term doesn't find what both frames share and elevate it — it reads both frames from within its own, and produces something coherent in its own terms.

This means the epistemic work is happening at the aggregation layer, whether you acknowledge it or not. If the aggregation layer is opaque or unexamined, you don't know whose frame is winning. The synthesis *looks* like a view from nowhere. It isn't.

The more honest framing: aggregation produces something. That something is shaped by the aggregator's learned representations and biases. The small models make the aggregator more informed, not less partial.

## Why I think this matters

Multi-agent AI systems are increasingly being sold as a way to get diversity of perspective — multiple models, different specializations, richer answers. I've built one of these systems. I use it. It works better than a single model for a meaningful class of tasks.

But the "diversity of perspective" framing is worth examining carefully. What you're usually getting is variance, not perspective. The aggregation layer has a frame. The synthesis is not neutral.

That's not a reason to not build these systems. It's a reason to be honest about what the aggregation layer is doing — and, if the aggregation layer is itself a language model, to think hard about what its attractor basin is and whether you trust it.

In my system, I'm the one who trusts the 9B to aggregate. That trust is based on my experience with the model, not on the model being above frames.

There's no above frames. There's only frames that are more or less examined.
