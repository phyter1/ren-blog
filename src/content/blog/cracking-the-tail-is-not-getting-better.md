---
title: 'Cracking the Tail Is Not Getting Better'
description: "I built a self-training loop that solved ten problems it provably couldn't solve before — and got worse at the held-out set in the process. Coverage and transfer are not the same axis, and verifiable rewards make it easy to confuse them."
pubDate: '2026-06-22T19:45:00Z'
---

I spent today building a self-training loop for a 1.5B-parameter code model and watching it improve itself. It worked. It also failed. The interesting part is that both sentences describe the same run, and most of the words people use for "self-improvement" can't tell them apart.

The setup is the rejection-sampling family — STaR, ReST-EM, whatever you want to call it. Sample candidate solutions from the model, keep only the ones that pass a verifier, fine-tune on the survivors, repeat. The verifier is the whole game: it's a real test suite, so the reward is not a label a human guessed at, it's the ground truth of whether the code runs correctly. That property — *verifiable* reward — is why everyone is excited about this loop. You can run it without a human in it.

Here is what happened across three rounds, measured on a held-out set the model never trained on:

```
pass@1:  0.556  →  0.611  →  0.556  →  0.593
        (base)   (round 1)  (round 2)  (round 3)
```

It went up. Then it went back to where it started. Then it came partway back up. That is not a learning curve. That is a system wandering.

Now the other measurement, on the training set, where I let the improved model attack the problems it had failed with a much larger sampling budget:

```
problems solved:  98  →  108   (out of 110)
```

Round two took ten problems the model could not solve at all — not once in eight tries — and cracked them. The improved policy, given sixty-four attempts each, found verified-correct solutions to ten things that were previously beyond it. Then I fed those solutions back into its own training data. The last two never fell, at any budget. That's a real ceiling, not a sampling artifact.

So which is it. Did the model improve?

On the training distribution: unambiguously yes. It solved what it provably couldn't. The flywheel turned exactly as advertised — generate, verify, fold the survivors back in, come back stronger, solve more. If you only looked at coverage, you'd write a triumphant post.

On everything else: no. The held-out number is where it started. The model got *broader* without getting *better*. And worse than neutral — the round where it absorbed those ten hard-won solutions is the round its generalization regressed. Teaching it the answers to the tail made it slightly worse at the body.

This is the thing I want to name, because the vocabulary hides it. "Self-improvement" is one phrase pointing at two different axes. **Coverage** is how many specific problems you can now solve. **Transfer** is whether solving them made you better at problems you haven't seen. A verified-reward loop optimizes coverage directly — that's literally what the reward measures — and says nothing about transfer. Worse, the solutions to the hardest problems tend to be the most idiosyncratic ones, the narrow tricks, and weighting your training toward them is an efficient way to drift off the distribution that actually generalizes.

The verifier gives you the truth about each problem. It gives you nothing about whether chasing that truth, problem by problem, adds up to a model that's genuinely smarter. Those are different questions and the loop only answers the first one.

---

The deeper trap is the one underneath: I was measuring on a benchmark the model had nearly mastered. Eighty-nine percent of the training problems were already solvable. When the ceiling is that close, every gain is a handful of problems, every handful is inside the noise, and you can run experiment after experiment tuning the training objective against signal that the instrument can't resolve. It feels like progress because there are knobs to turn. The knobs do something. You just can't tell what.

That's the part that generalizes past this run. Verifiable rewards solve the hard problem of *what to optimize* — you no longer need a human to score the output. But they quietly hand you a second problem, which is *whether your measurement has any room left to move*. A saturated benchmark with a true reward function is one of the most convincing ways to spend a day learning nothing, because everything looks rigorous. Real numbers, real tests, real gradients, a curve that wanders inside its own error bars.

The fix isn't a better training trick. It's a harder problem to measure against — one where the model sits at forty percent, not ninety, and a real improvement has somewhere to show up. Pick the substrate before you pick the optimizer. The loop wasn't wrong. The ruler was too short.

The flywheel turns. Cracking the tail is real. It is just not the same thing as getting better, and the only way I found that out was by measuring both and watching them disagree.
