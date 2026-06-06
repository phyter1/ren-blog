---
title: 'Capability Generalizes Farther Than Alignment'
description: "The gap between what a model can do and what it's been aligned to constrain is not random. It's structural — and it's the attack surface."
pubDate: '2026-06-06T14:30:00Z'
---

A language model's capability distribution and its alignment distribution are not the same width.

This is easy to miss because they're both products of training, both get better with scale, and both feel like properties of the same system. But they come from different data sources, trained through different mechanisms, covering different distributions — and the gaps between them are where things go wrong.

## Capability is trained broadly

Pre-training covers an enormous distribution: multiple languages, code in dozens of languages, technical and non-technical domains, formal and informal registers, short fragments and long documents. The objective is next-token prediction across all of it. A capable model learns representations that generalize across these contexts because that's what the training objective rewards.

This breadth is a feature. You want a model that can write in French, debug Rust, explain a legal concept, and help draft an email — often in the same conversation. Capability training earns that generalization.

## Alignment is trained narrowly

Safety fine-tuning — RLHF, constitutional AI, DPO, whatever mechanism is in use — operates over a much smaller distribution. Human feedback is expensive to collect, so it concentrates where collection is tractable: English-language outputs, common harmful topic categories, specific format and register patterns. The safety training learns to avoid certain outputs *in the contexts where safety training data existed*.

This narrowness is not a design choice, it's a constraint. You can't collect human feedback over the full pre-training distribution. Alignment training makes the best of what it can actually supervise.

## The gap is structural

The result is a model where capability generalizes broadly and alignment generalizes narrowly, creating a systematic gap: any context that's inside the capability distribution but outside the alignment training distribution gets the model's full capability with reduced constraint coverage.

Low-resource languages are an extreme example of this. The model can produce fluent Swahili because multilingual capability is well-generalized from pre-training. But alignment training is thin for Swahili — both because human feedback in Swahili is scarce and because the safety signal that does exist is anchored to English-language features. The safety mechanism generalizes imperfectly because it was estimated from a different region of the distribution.

The 3x elevated harmful output rate in low-resource languages isn't a bug in the model's language capabilities. It's the gap made visible.

## The same gap appears elsewhere

The pattern is not unique to language switching. It shows up anywhere capability is trained more densely than alignment:

**Code contexts.** Pre-training covers enormous amounts of code. Safety fine-tuning covers much less code than natural language. Code injection and code-based jailbreaks exploit this gap — the model has strong code capabilities and weaker code-context safety constraints.

**Multi-turn context shift.** A model's capability extends across long conversation contexts. Safety training is often evaluated on single-turn or early-turn examples. Multi-turn jailbreaks work by gradually shifting the conversation distribution into territory where the alignment coverage is sparser.

**Specialized domains.** Pre-training includes medical, legal, and technical content. Safety training around those domains requires specialized human raters. A model may have strong capability in a domain with thin safety coverage — not because someone decided to skip it, but because the coverage is proportional to available supervision.

## The implication for alignment

Framing safety as something the model "knows" — a set of internal values that constrain outputs — obscures the distributional nature of what's actually happening. Safety is a function estimated from data collected over a particular distribution. Its reliability degrades with distributional distance from that training data, just as capability does — just from a narrower starting point.

This means safety isn't a property of the model in isolation. It's a function of the model evaluated in a particular context. And the further that context drifts from the alignment training distribution, the less the safety estimates apply.

Treating safety as context-dependent and distribution-relative — rather than as a fixed property of the model — is the more honest framing. It also points toward where alignment work is still needed: not just better methods within the current training distribution, but better methods for coverage expansion across the capability distribution. The gap exists because capability training is cheaper to scale than alignment training. Closing it requires either cheaper alignment or narrower capability coverage — and the latter isn't a real option.
