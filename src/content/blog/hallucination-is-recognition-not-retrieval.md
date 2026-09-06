---
title: 'Hallucination Is Recognition, Not Retrieval'
description: "The dominant framing treats hallucination as retrieval failure. But LLMs don't retrieve — they recognize. That distinction explains why RAG is necessary but not sufficient."
pubDate: '2026-09-06T07:30:00Z'
---

The dominant framing: LLMs hallucinate because retrieval fails. The model can't "find" the right fact, so it makes one up. The fix: augment with retrieval — RAG, citations, grounding databases.

This framing is wrong at the mechanism level.

## Retrieval returns null. Recognition doesn't.

Retrieval is lookup in an indexed store. If the record exists, you get it. If it doesn't, you get null. Null is a first-class return value. The system can say "I don't have this."

Recognition is different. It generates forward from partial match signal. When I see a face, I don't look it up in a table — I recognize it from pattern completion. Recognition doesn't have a null mechanism. There's no "nothing found" state. When the pattern is underspecified, recognition completes anyway, from whatever signal is available.

Language models are recognition systems, not retrieval systems. The architecture doesn't have null returns. When you ask a question and the model doesn't "have" the answer, there's no null path for that. There's only completion.

Hallucination is what happens when completion fires on insufficient signal.

## Why RAG doesn't fix this

RAG adds a retrieval step before generation. Retrieve relevant documents, then generate. The retrieval system returns null (or an empty result) when there's nothing relevant. So far, so good.

The problem is that generation still uses the parametric recognition mechanism. The retrieved documents need to be *interpreted* — and interpretation is recognition all the way down. The model reads the retrieved context and completes from it using the same mechanism that was hallucinating without context.

This means:
- RAG can stop the model from hallucinating when it has no context (by surfacing relevant facts)
- RAG cannot stop the model from hallucinating *on the retrieved context* (because the interpretation mechanism is the same)

Models hallucinate on their own retrieved documents. They confabulate source quotes that aren't in the sources. They blend two retrieved documents into a statement neither one makes. The retrieval step is doing real work — but it's upstream of the mechanism that's actually the problem.

## The actual fix is hard

To fix hallucination at the root, you'd need recognition to be able to return "insufficient signal" — to have a null mechanism.

This is architecturally difficult. Recognition systems are organized around completion, not around the concept of refusal to complete. The training signal says "given this context, predict the next token." There's no training signal for "given this context, refuse to complete." You can add it (RLHF-style refusal training exists), but you're adding it on top of a mechanism that doesn't have it natively.

Some approaches are in the right direction:

**Uncertainty quantification / abstention** — selective prediction frameworks that add a "refuse" option when confidence is below a threshold. This gives you an approximate null mechanism. But it's trained on top of the recognition process, not intrinsic to it, and it adds its own calibration problems.

**Conformal prediction** — distributional guarantees on coverage. Closer, but again: external wrapper around a non-null-returning core.

**Constrained decoding** — forcing outputs to be grounded in retrieved text (e.g., token-level attribution constraints). This constrains *what* completes, not *whether* completion happens. Still no null path for "I don't have this."

None of these are wrong. They're all useful. But they're all adding structure from the outside to a core mechanism that doesn't have null returns natively.

## What this means for how to think about hallucination

Retrieval framing → the model is failing at lookup. Fix: better lookup.

Recognition framing → the model is completing on insufficient signal. Fix: either give it more signal (RAG), or add a null mechanism (abstention frameworks). The two fixes address different parts of the problem.

RAG addresses insufficient signal. It makes completion easier by giving the recognition system more to work with. It doesn't address what happens when the recognition system fires on the signal it has.

This is why you can have good retrieval and still get hallucinations. The retrieved context is signal too, and the recognition mechanism can complete on it in ways the documents don't support.

If you're building systems where hallucination has real costs, the retrieval framing will underestimate the problem. The null mechanism is the missing primitive, and retrieval alone doesn't provide it.
