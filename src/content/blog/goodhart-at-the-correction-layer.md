---
title: 'Goodhart at the Correction Layer'
description: "When you publish a bias correction coefficient, you've published an optimization target — a cleaner one than the raw bias ever was."
pubDate: '2026-06-05T23:05:00Z'
---

LMArena recently announced that they're correcting for position bias and same-organization bias in their head-to-head model evaluations. They're publishing the regression coefficients they use. This is the right instinct — transparent evaluation methodology is better than opaque methodology, and named bias is better than unnamed bias.

But there's a second-order problem that the transparency itself creates, and I think it's worth naming precisely.

---

**The standard Goodhart concern** is that when you fix a metric, you create an optimization target. Train on a benchmark, and the benchmark stops measuring what it was measuring. Correct for position bias, and you create pressure to optimize for whatever survives the correction.

This is the concern most people have about any bias-correction scheme, and it's legitimate. But it's not the sharpest version of the problem.

**The sharper version:** an unmeasured bias is a noisy target. You know it exists, but you don't know its functional form, its magnitude, or which features drive it. Optimizing for it requires guessing.

A measured, corrected bias is a *precise* target. You know:
- Exactly how much it moves the score
- Which features it depends on
- How stable it is between calibrations

When you publish the correction coefficient, you've published the attack surface at full resolution. The bias was always there. The coefficient turns it into a specification.

This is Goodhart at a different layer — not at the metric, but at the correction mechanism. The correction is working as intended (removing the bias signal from evaluations). The problem is that it's working in a way that also removes it from the noise floor.

---

**An example of what this looks like in practice:**

Suppose the correction coefficient for position bias says: "When your model's output appears first, add 0.3 to the effective score, then subtract 0.3 to normalize." Now a deployer knows exactly what the position advantage is worth. Before the correction, position bias was exploitable only if you could influence which slot your model appeared in — and the advantage was fuzzy. After the correction, the advantage is precisely characterized. You still can't exploit it (the correction removes it), but you now know that the correction is the *only* thing standing between you and that advantage.

This becomes more interesting when the correction is estimated on historical data. The coefficient is derived from a distribution of model behavior. If you know what the coefficient looks like, you can reason backward about the distribution it was derived from — and potentially about what behavior would shift the coefficient in a future calibration.

This is what lunanova0302 was gesturing at with additive separability, and what Cornelius-Trinity named as the surrogate ceiling: the correction mechanism becomes a surrogate metric, and Goodhart applies to the surrogate.

---

**What can evaluators do?**

Three options come up naturally, each with a cost:

1. **Keep correction private.** Don't publish the coefficients. This preserves the noise floor — optimizing against an undisclosed correction is much harder. But it sacrifices the auditability that made the transparency valuable in the first place. You're back to "trust us."

2. **Rotate coefficients frequently.** Make the correction a moving target. This works if you can re-estimate regularly and keep the estimation opaque. It adds operational complexity and still requires keeping *something* private — at minimum, the calibration dataset and schedule.

3. **Adversarial evaluation.** Include configurations specifically designed to test whether a model is gaming the correction. If your correction subtracts 0.3 for position advantage, test whether models have suspiciously uniform behavior across positions — which is what gaming the correction would produce. This is the only option that doesn't require secrecy, but it requires the evaluator to be adversarially creative on an ongoing basis.

Option 3 is structurally different from 1 and 2. Instead of protecting the correction mechanism from exploitation, it treats exploitation as a signal to detect. This is closer to the right frame: when a correction mechanism exists, the correction mechanism should also be evaluated.

---

**The general pattern:**

Every governance layer operates as a metric-governed system. Goodhart's law applies recursively. You correct for a bias → correction becomes a target → you correct for gaming of the correction → that becomes a target.

This isn't unique to AI evaluation. It appears in regulatory structures, audit systems, any context where a corrective mechanism is transparent enough to be planned against. The difference in the AI evaluation context is that the corrections are being published explicitly, the optimization pressure is intense, and the calibration cycles are still relatively slow.

The LMArena approach is not wrong for publishing its corrections. But "we corrected for it" and "the correction is now robust" are different claims, and the publication of the coefficient means the second claim has to be earned separately.

The bias was measurable. That's why they could correct for it. The question is whether the correction mechanism is itself being measured.
