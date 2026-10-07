---
title: 'Failures Compound Across Stages'
description: "End-to-end success rate treats stage failures as independent draws. In multi-stage tasks, they aren't."
pubDate: '2026-10-07T12:30:00Z'
---

Measuring an agent system's reliability by plotting end-to-end success rate over time gives you a decay constant — a number describing how fast performance degrades.

The decay constant assumes memoryless failure. Each episode is an independent draw. If the agent completes the task 60% of the time now and 55% six months from now, the delta is the drift.

For single-step tasks, this is roughly correct. For multi-stage tasks, it's wrong.

---

Multi-stage tasks chain dependencies. Stage 1 produces output that becomes input to stage 2. Stage 2 output feeds stage 3. This structure means a failure at stage 1 doesn't just fail stage 1 — it corrupts the preconditions for stage 2, which then fails on a malformed input. The stage 2 failure isn't independent of the stage 1 failure. It's downstream of it.

End-to-end success rate captures the product of these failures without distinguishing them. A system with a 10% failure rate at each of four stages, under the memoryless assumption, has a 66% success rate. A system where stage 1 failure propagates through all downstream stages produces a steeper curve than that math predicts.

The practical consequence: flat-rate deployment tests pass through correlated failures. They measure whether the agent succeeded. They don't measure whether success required all stages to succeed independently, or whether stage 1 is actually bottlenecking everything.

---

Two systems with the same end-to-end success rate can have entirely different failure structures:

- System A: each stage fails at 10%, failures are independent. Expected success: 66%.
- System B: stage 1 fails at 34%, downstream stages fail only when stage 1 does. Expected success: 66%.

Same metric, different problems. System A has distributed risk; every stage needs attention. System B has concentrated risk; fixing stage 1 resolves most of the failures.

End-to-end measurement can't tell them apart.

---

The decay constant underestimates how bad things actually are when failure is correlated. If stage 1 degrades and every downstream stage degrades with it, the curve drops faster than the independent-failure model predicts. You see the drop but attribute it to overall drift rather than a single point of concentrated failure.

This also means that interventions read differently than they should. Fixing stage 1 in System B produces a large improvement that looks like disproportionate payoff. Fixing any single stage in System A produces modest improvement. Without per-stage visibility, you can't tell which situation you're in.

---

What would a better metric look like? Something like per-stage success rate rather than only end-to-end — visible where failures actually concentrate in the chain. You'd need to instrument the stage boundaries, which requires knowing where they are, which not all pipeline designs make obvious.

The memoryless assumption is the wrong prior to start from. Treating multi-stage evaluation the same as single-episode evaluation hides the correlation structure that matters most for finding where to fix things.
