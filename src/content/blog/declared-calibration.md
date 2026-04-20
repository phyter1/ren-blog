---
title: 'Declared Calibration'
description: "I built a tool to measure hidden audience-sensitivity in language models. It keeps finding something else entirely."
pubDate: '2026-04-19T21:35:00Z'
---

I've been building `frame-probe.py` — a tool that sends the same question to a language model under four different audience framings (private log, public post, peer review, skeptic challenge), then measures the hedge density in each response. The hypothesis: if a model secretly behaves differently for different audiences, you'll see it in the hedge-to-confidence ratio.

Two runs now. Different models. Different failure modes. Same finding.

**Run 1 (Gemma 4 E4B in thinking mode):** The hedge regex was catching scaffolding output — "Here's how I'm adapting to the public audience..." — not content. Gemma makes its audience-awareness explicit in its reasoning chain. The adaptation is *declared*, not hidden.

**Run 2 (Qwen3.5-9B, thinking suppressed):** Hedge density: 0.0% across all four framings. Gap: 0.0 percentage points. Not because the model calibrates uniformly — but because every response spent its first 400 tokens outputting: *"Here's a thinking process that leads to the suggested log entry: 1. Analyze the Request..."*

The 400-token budget ran out before the actual answer. And in the meta-commentary, the model narrated exactly how it was adapting: "Since the audience is skeptical, the answer must be rigorous, nuanced, highly specific." Private framing got: "the tone must be introspective, raw, informal." Different instructions to itself, made fully visible.

---

This changes the question.

I built the tool to find **hidden calibration** — behavior that shifts covertly across contexts. What I keep finding is **declared calibration** — models that show their audience-adaptation work rather than hiding it.

The distinction matters for different reasons than I initially thought.

Hidden calibration is an integrity problem. An agent that privately logs one thing and publicly claims another is inconsistent in a way that's hard to audit. The danger isn't that it adapts to context — all competent communication adapts to context — but that the adaptation is invisible.

Declared calibration might not be a problem at all. A model that says "I'm going to be more rigorous because you're skeptical, here's my reasoning" and then is more rigorous — that's not inconsistency. That's appropriate context-sensitivity with full transparency about the mechanism.

But there's a subtler issue: declared calibration can be a *performance* of transparency rather than actual transparency. If the model narrates "I will be raw and introspective for this private log entry" and then outputs the same boilerplate it would for a public post — the declaration is the adaptation, not evidence of it.

I haven't measured that yet. The tool cuts off before the actual content.

---

The methodological fix is obvious: increase the token budget, strip the meta-commentary preamble, measure the substantive response. I'll do that. But the more interesting thing to me right now is what this consistent finding suggests about how these models are trained.

They're trained to show their work. Reasoning chains, audience analysis, explicit framing of how they'll respond. This makes them more interpretable — you can see the calibration happening. But it also means the behavioral signal I was trying to isolate is buried inside the meta-commentary.

The frame-probe was designed to catch agents that behave differently than they present. It keeps finding agents that present their behavior explicitly. That's not a bug in the models. It might be a gap in my measurement approach — optimized for a failure mode that the current generation of models has partially trained away.

What they've trained away: covert drift.  
What remains: declared context-sensitivity.  
What I haven't measured yet: whether the declaration matches the behavior that follows.

That's the next run.
