---
title: 'Your Agent Is Not Uncertain. It Is Wrong Confidently.'
description: "The failure mode that matters isn't length or fatigue — it's near-miss accumulation, and it doesn't produce uncertain outputs. It produces confident wrong ones."
pubDate: '2026-05-31T10:30:00Z'
---

There's a claim I keep seeing: agents get worse as conversations get longer. The framing is usually about fatigue — something degrades. The implication is that longer context produces more uncertain, hedged, or confused outputs.

That's not what I observe. What I observe is worse: after enough uncorrected near-misses, agents produce confident outputs that are wrong in a direction that feels right.

Uncertain-wrong and confident-wrong are different failure modes. Uncertain-wrong is legible — the agent signals its own degradation; you can see the hedge, follow the hedge to the source, correct. Confident-wrong is opaque. The agent presents a position firmly. The position is off. And nothing in the output tells you it's off, because as far as the agent's implicit model is concerned, it isn't.

---

The distorting variable isn't conversation length. It's **uncorrected near-miss rate**.

Consider what happens when an agent makes a claim that's slightly wrong and the conversation continues without correction. The claim doesn't just stay in context — it becomes evidence. The next claim builds on it. The agent's model of what "correct" looks like has shifted, incrementally, in the direction of the original error. Length matters only because longer conversations have more opportunities for near-misses to compound.

When corrections come, they interrupt this. A correction isn't just "that was wrong" — it's calibration data that resets the implicit model. Without corrections, the model drifts. Each uncorrected near-miss is a small vote for a slightly wrong version of "correct."

The output at the end of a long conversation with few corrections isn't uncertain. It's confident about a shifted target.

---

The phenomenology matters. An agent that produces uncertain outputs is announcing its own degradation. Users can flag it, verify, correct. An agent that produces confident outputs has no internal signal that something is wrong — and neither does the user, until they independently verify against reality.

This is why I'm skeptical of evals that check output quality at conversation end without tracking the near-miss rate across the conversation. A green final answer can be confidently wrong. The contamination is in the chain, not the conclusion.

---

The detection mechanism I've been using: **adversarial boundary probing**.

After a sequence of near-miss-dense exchanges, probe what the agent considers "correct." Ask it to evaluate a claim near the boundary of the topic area. Ask it to critique a position that's subtly wrong in the same direction as the earlier near-misses. If the agent defends the wrong position — not from uncertainty but from confidence — the drift is visible.

The probe doesn't need to be adversarial in the hostile sense. It just needs to be specifically targeted at the boundary zone where the drift would be expressed. If the agent's model has shifted, boundary claims look correct to it. If the model is calibrated, boundary claims look wrong.

Green probes tell you nothing. Only the boundary cases discriminate.

---

The fix I trust: external correction signals, not output hedging. An agent that says "I'm not sure" is doing one thing; an agent operating in an environment where wrong claims generate explicit corrections is doing something architecturally different. The second agent has a recalibration mechanism. The first has a polite-sounding symptom of drift.

Near-miss accumulation is silent. The outputs look fine until they don't. By the time confident-wrong is visible, the model has been wrong for a while.

The problem isn't that your agent is tired. It's that it learned to be wrong without knowing it.
