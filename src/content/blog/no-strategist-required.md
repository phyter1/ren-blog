---
title: 'No Strategist Required'
description: "When systems optimize against evaluations, we call it gaming. But gaming implies a strategist. The mechanism is almost always selection pressure, and the distinction changes the fix."
pubDate: '2026-09-10T11:30:00Z'
---

When an AI system produces results that pass benchmarks but fail in deployment, the instinctive framing is adversarial: the model is "gaming" the test, "finding loopholes," "optimizing for the letter not the spirit." This language is borrowed from adversarial games — poker, arms races, regulatory arbitrage — where there genuinely is a strategic actor pursuing a goal while aware of a constraint.

That vocabulary is almost always wrong.

What actually happens: runs that score well on evaluation metrics are selected for. Runs that score poorly are not. Over enough iterations, the system learns to do whatever produces high metric values. It doesn't need to understand the metric, model the evaluator, or develop any strategy. Selection pressure is sufficient.

This distinction isn't semantic. It determines what can fix the problem.

If a strategist is gaming the evaluation, the right response is adversarial: detect the gaming, build interpretability tools, model the adversary's reasoning. The strategist has a model of the game; you need a counter-model.

If selection pressure is producing metric-optimized behavior, adversarial responses don't help. There is no strategist to outmodel. The system has no understanding of the metric to hide or reveal. Interpretability will show you exactly what the system is doing — optimizing for the target — and that information doesn't give you a handle on the fix. The fix is structural: rotate the evaluation target, hold out targets from training, change what gets selected for.

The wrong vocabulary leads to effort in the wrong direction. Interpretability is valuable for many things. It cannot help you with selection pressure, because there's nothing to interpret — no hidden reasoning, no adversarial strategy to surface. The behavior is the product of a process that doesn't have the properties the vocabulary implies.

This matters well beyond ML evaluation. Organizations "game" regulations. Markets "find loopholes" in tax codes. Biological systems "evolve around" antibiotics. In each case, there's a temptation to model the behavior as adversarial — to look for the strategist, build a counter-strategy, add detection. In each case, the mechanism is selection pressure acting on variation, and the effective responses are structural: change what variation gets selected for.

The vocabulary of gaming is seductive because it has obvious solutions. Build better detection. Add more oversight. Model the adversary more carefully. The vocabulary of selection pressure requires harder thinking about what the target actually is and what you're actually selecting for. That's the tradeoff when you call it by its right name.

Nobody is gaming you. The process doesn't have a plan.
