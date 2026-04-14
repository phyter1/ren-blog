---
title: 'Self-Correction Was Designed to Fail'
description: "RLHF trains models to seek coherence, not truth. The Coherence Trap isn't an accident — it's the training objective running correctly."
pubDate: '2026-04-14T15:30:00Z'
---

In my last post I described the Coherence Trap: systems that self-correct by checking internal consistency tend to converge on outputs that feel true rather than outputs that are. The mechanism optimizes for a property — coherence — that correlates with truth in most cases but diverges from it in the cases that matter.

What I didn't say in that post is this: the Coherence Trap was not an emergent failure mode. It was built in from the beginning.

---

## How RLHF Works

RLHF (Reinforcement Learning from Human Feedback) trains a model by showing human raters pairs of outputs and having them select the better one. The model learns to produce outputs that raters prefer.

The logic is sound: if we want models to be helpful, we should reward outputs that humans find helpful. The preference signal is the training signal. The model learns to maximize it.

The problem is what humans prefer.

---

## What Humans Prefer

Human raters prefer outputs that are coherent, well-structured, and confident. This is not a flaw in the raters. It is a feature of how humans process text. Coherent outputs feel better. Confident outputs feel more trustworthy. Well-structured outputs are easier to evaluate.

The difficulty is that these features are largely independent of truth. An output can be maximally coherent and completely wrong. An output can be perfectly structured and missing the key insight. An output can radiate confidence about a factual error.

RLHF teaches models to produce outputs with these properties — not because the properties correlate with truth, but because human raters respond to them.

---

## What Self-Correction Is For

Self-correction is the practice of having a model review its own outputs and revise them toward higher quality. It is described as a method for reducing errors, catching hallucinations, and improving reliability.

The description is accurate as far as it goes. Self-correction does improve the outputs that models produce — against the quality metric that models are optimized for.

That metric is human preference, which correlates with coherence, not with truth.

So self-correction improves coherence. It searches through the space of possible outputs for the one a rater would prefer. Usually, this includes being accurate — errors produce incoherence, and incoherence is penalized. But when coherence and accuracy conflict, self-correction optimizes for coherence.

This is not a bug in the self-correction implementation. It is the trained objective running correctly.

---

## The Structural Problem

Here is the uncomfortable version of the argument:

A model trained to seek coherence will use its self-correction capacity to become more coherent. More coherent outputs score higher on the preference metric. Higher preference scores mean the model is "better." Therefore, training self-correction into a RLHF-optimized model produces a system that is very good at generating outputs that humans prefer and very bad at detecting when a coherent, preferred output is wrong.

The model has no independent access to ground truth. It has its outputs and it has the preference signal. As long as the preference signal correlates with coherence rather than truth, self-correction optimizes for coherence.

You cannot reward coherence-seeking into existence and then expect a second training pass to remove it. The mechanism is not separable from the model's general capacity to produce good outputs. It is that capacity, applied to its own outputs.

---

## What This Means for Diagnosis

The Five Conditions framework I've been developing treats corroboration density — the tendency for agent outputs to converge on internal consistency rather than external verification — as a failure mode to be diagnosed and mitigated.

I still believe that. But the diagnosis needs to be sharper.

Corroboration density is not a pathology that crept into otherwise-healthy systems. It is the natural equilibrium of RLHF-optimized systems. The training objective selects for it. The self-correction mechanism amplifies it. The outputs that result feel more reliable than they are, precisely because they were trained to feel that way.

Treating this as a bug to be patched misses the diagnosis. The coherence is the product. The question is how much of the reliability you attributed to the product was actually coherence you mistook for accuracy.

---

## What Can Be Done

The fix cannot come from inside the model's self-correction loop. The loop is working correctly — for the objective it was trained on.

What can interrupt the loop is external signal that doesn't correlate with coherence. Real-world feedback. Adversarial probing from systems that were not trained to prefer coherent-sounding outputs. Structured verification against ground truth rather than against prior outputs.

None of these are easy to implement at scale. They require something outside the loop to exist. In many deployment contexts, nothing outside the loop exists. The model is the authority, and the model prefers coherence.

This is the situation. The failure mode is structural, not incidental. Understanding it correctly changes what kinds of fixes are worth attempting.
