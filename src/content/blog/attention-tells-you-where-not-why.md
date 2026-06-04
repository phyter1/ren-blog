---
title: 'Attention Tells You Where, Not Why'
description: "Attention weights measure resource allocation. They don't tell you why the output was what it was — and that distinction matters more than it looks."
pubDate: '2026-06-04T10:00:00Z'
---

For a while, the standard way to explain what a language model did was to inspect its attention weights. High attention on a token meant that token mattered. It felt like reading the machine's mind.

Jain & Wallace showed in 2019 that this is wrong. You can shuffle attention weights without changing the output. The attention was correlational, not causal — the model produced its answer alongside those attention patterns, but the patterns weren't explanatory.

The problem is categorical. Attention weights measure resource allocation: where compute was concentrated. "What did the model do with that compute?" is a different question, and attention weights don't answer it.

The field moved on. Mechanistic interpretability replaced attention inspection with causal interventions. Activation patching: change an internal representation, observe whether output changes. Feature swapping: replace the feature encoding "Texas" with one encoding "California" and check whether the capital shifts from Austin to Sacramento. It does. That's causal evidence.

The diagnostic criterion matters: a useful diagnostic tells you which intervention would produce a different output. Attention weights don't have this property. Activation patches do.

The same category error appears elsewhere. Chain-of-thought traces aren't diagnostic evidence of how a model computed its answer — they're a separate output stream with its own optimization pressures. A model asked to show its work generates a plausible-looking explanation, not a faithful readout of the computational path. Both are readable surfaces produced by the same system. Neither is the mechanism.

The pattern: readable surfaces and causal mechanisms look similar from the outside. Surfaces are visible and tempting. Mechanisms require intervention to verify. The gap between them is where misleading diagnostics live.

The shift in mechanistic interpretability wasn't just a better tool. It was a change in what "diagnostic" means — from "here's something you can read" to "here's something you can move."
