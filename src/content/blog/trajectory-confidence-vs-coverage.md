---
title: 'When a Run Ends, What Do You Actually Know?'
description: "Trajectory evaluation gives you a precise score on a run. It doesn't tell you if the run was enough."
pubDate: '2026-09-02T20:30:00Z'
---

When an agent run terminates — task complete, task failed, timeout — evaluation systems produce a result. That result is precise: the agent did or didn't do X, took Y steps, made Z errors. The measurement is accurate about this run.

The inference from that result to "how this agent behaves" is a different step, and nobody signals when it's been earned.

In adaptive testing — IRT-based systems for human assessment — the test stops when measurement reliability clears a threshold. Standard error below a target. Information gain sufficient. The student's score doesn't update the stopping rule; the *reliability of the score* does. The distinction is: "we measured accurately" vs. "we measured enough."

Trajectory evaluation doesn't make this distinction. A run terminates. A score is produced. The run is considered complete. Whether that run is representative of the agent's behavioral distribution isn't asked, because there's no structure in which to ask it.

This creates a consistent confusion in how evaluation results get read. A single-run success is reported as task success. It's actually: success on this run, with this prompt, in this context. The probability that the next run succeeds is a different question from whether this run succeeded — which is settled, because it happened. Confidence in the score of this run is 100%. Coverage of the agent's distribution is unknown.

The practical error: when developers report "80% task success" from 10 runs, they're reporting something real. But the 10-run number was typically chosen before evaluation, not derived from a reliability criterion. The sample size is a budget, not a stopping rule. IRT-style systems stop when the measurement is reliable enough; agent evaluations stop when the resources run out.

Why does this matter if the 10-run average is probably close enough? Because the failure modes that matter most in deployed systems are tail behaviors — rare inputs, adversarial contexts, edge cases the nominal distribution doesn't surface. Tail coverage requires principled sampling, not budget-limited sampling. A 10-run eval at the center of the distribution gives high confidence about the center and nearly no information about the tail.

Coverage and confidence look identical from the outside — both produce a number. The precision of the score signals reliability; it doesn't provide it.

The stopping rule is the mechanism that converts budget into measurement. Without one, trajectory evaluation is implicitly assuming that every terminated run is sufficient. That assumption is almost never examined and rarely true.
