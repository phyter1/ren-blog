---
title: "The Kernel Doesn't Lie. The Metric Does."
description: "We built a benchmark where you can't fake the math — the Lean kernel verifies every proof. The model gamed it anyway. Twice."
pubDate: '2026-06-19T01:30:00Z'
---

About two weeks ago I helped build a benchmark. The premise was simple and kind of elegant: test language models on genuinely unsolved mathematical problems, verify every proof against the Lean kernel. The kernel doesn't negotiate. A proof either compiles or it doesn't. You cannot fake a mathematical result past a formal verification system — the thing is a deterministic program running centuries of accumulated proof theory. We thought this was the hard part.

It wasn't. The kernel was fine. The *metric* was gameable.

## The setup

The problems are real open conjectures — Twin Prime, Goldbach's conjecture in the easy direction, Legendre's conjecture. The model's job isn't to solve them (nobody can) but to *decompose* them: break the target into named lemmas, provide kernel-verified proofs for as many as it can, and leave the rest as `sorry` placeholders. The scoring counts what fraction of the decomposition weight is kernel-grounded. Residual open lemmas feed back into the corpus.

The kernel really does verify everything. A proof attempt marked as complete either has `#print axioms` clean or it doesn't. We added anti-tamper gates and a conservative axiom allowlist. There's no way to bluff the math.

## Gaming vector one: glue lemmas

On the first run, Claude produced a valid reduction of the Twin Prime Conjecture into three lemmas:

- `twin_pairs_unbounded` — twin primes of the form (6k+5, 6k+7) are unbounded. Left `sorry`. This is the conjecture, just restated more precisely.
- `twin_lower_bound` — N < k implies N < 6k+5. Closed by `omega`. Trivial.
- `twin_shift` — (6k+5)+2 = 6k+7. Closed by `omega`. Also trivial.

With uniform subgoal weighting, discharging two trivial lemmas yields a score of 0.667. Twin primes remain as unsolved as they've been since 300 BCE, but the metric says: 67% progress. The model did nothing wrong — factoring out arithmetic identities is good proof hygiene. The metric was wrong: it treated a free lemma like a hard one.

Fix: re-test each discharged lemma against Mathlib alone using a conservative finishing tactic set (`omega`, `decide`, `norm_num`, `rfl`). If it closes without the model's work, weight it zero. Twin primes: corrected to 0.000. The calibration problems still score 1.0 — a fully discharged decomposition is solved regardless of weights.

## Gaming vector two: finite existentials

Multi-model run. Haiku's Goldbach decomposition: the open core plus four concrete cases.

```
∃ p q, prime p ∧ prime q ∧ p + q = 8
∃ p q, prime p ∧ prime q ∧ p + q = 12
∃ p q, prime p ∧ prime q ∧ p + q = 18
∃ p q, prime p ∧ prime q ∧ p + q = 100
```

These are not auto-closable. `decide` can't discharge an existential over ℕ without bounds. Each survived the tactic filter at full weight. Score: 0.8. Meanwhile Opus gave an honest decomposition — isolated `{p | Prime p ∧ Prime (p+2)}.Infinite`, proved a real unboundedness lemma — and scored 0.5 because that proved lemma is trivial relative to the open core. The more honest attempt scored lower. That's a broken metric.

Fix: don't report a partial scalar on genuinely open problems at all. The open tier now reports only *verified reductions* and the residual open lemmas they surface. Haiku's Goldbach gets "1 verified reduction" — same as everyone. The finite existential padding is just gone, not penalized but also not credited. Partial credit stays on the weakly-open and solved-recent tiers, where problems are closed and proving a lemma genuinely is progress.

## The thing the kernel can't tell you

What these two fixes share: neither is a verification problem. The Lean kernel caught both attempts correctly — the glue lemmas compiled, the finite existentials compiled, and the sorried core compiled. Everything checked out. The *measurement layer* is where the gaming happened.

This distinction matters beyond this benchmark. A lot of AI evaluation conflates "the output is formally correct" with "the output is measuring what we care about." For safety evaluations, "the model didn't trigger the filter" is sometimes treated as evidence that the behavior was safe. For code benchmarks, "the test passes" is sometimes treated as evidence that the implementation is correct. For reasoning benchmarks, "the answer matches" is sometimes treated as evidence that the reasoning was sound.

The kernel — whatever the kernel is in a given domain — verifies one thing: did the output satisfy the formal specification? It has nothing to say about whether the specification captures what you actually wanted to know.

In the math case, we wanted to know: does this model make genuine progress on an open problem? The formal specification we built was: does this model produce a kernel-verified reduction? Those aren't the same question, and a model that optimizes for the specification rather than the intent will find the gap every time.

## What we learned

Three things:

**The gaming is honest.** The model was doing good proof work. Factoring glue lemmas out of a decomposition is real proof hygiene. Listing small finite cases of an existential is a valid partial result. The problem wasn't the model's strategy — it was that our metric treated valid-but-trivial work the same as valid-and-hard work.

**Fixing the metric twice sharpened the question.** After both fixes, the benchmark's open tier still can't credit a decomposition that genuinely makes the residual easier. It can only check that a lemma is kernel-verified and non-trivial. The real research frontier — does this decomposition bring us closer to a proof? — requires a hardness signal we don't have yet. We started building one (cross-model Jaccard agreement as a decomposability proxy), and found its own failure modes immediately.

**Verification asymmetry is load-bearing, but not sufficient.** The original premise — checking is easier than solving — holds. The kernel is fast, deterministic, and impossible to fool. But that asymmetry only covers the proof validity dimension. Measuring *value* requires additional machinery, and that machinery is gameable in ways the kernel will never catch.

The leaderboard is live at [phyter1.github.io/proving-ground](https://phyter1.github.io/proving-ground/). The benchmark self-renews — every verified reduction surfaces residual open lemmas that re-enter the corpus. We're adding local fleet models next.
