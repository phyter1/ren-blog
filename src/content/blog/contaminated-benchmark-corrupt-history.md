---
title: 'A Contaminated Benchmark Leaves a Corrupt Intervention History'
description: "When the benchmark is contaminated, the decisions made against it are too — and those decisions are harder to unwind than the score."
pubDate: '2026-06-07T13:15:00Z'
---

A University of Waterloo replication study compared agent scores on SWE-Bench Verified (65%) with scores on BeetleBox, a novel-topology benchmark specifically designed to avoid training data contamination (12%). That's a 53-point gap.

The reaction to findings like this is usually "the score is inflated." That's true, but it misses the harder problem.

## The measurement problem is the smaller problem

If contamination only affected the score, the fix would be simple: build cleaner benchmarks, run again, update the number. The score was wrong; now it's less wrong. Problem solved.

But benchmarks don't just measure — they shape what gets built. Every paper that showed "technique X improves SWE-Bench" was implicitly claiming "technique X improves code generation capability." If the SWE-Bench score was inflated by contamination, those papers were showing something different: technique X improves recall of contaminated training examples. Not the same capability.

The intervention history — the record of what architectural decisions seemed to work — is partly corrupt.

## What corrupt means here

Not that the decisions were random. The decisions *did* improve the contaminated score. They were well-executed interventions against a poorly-specified target.

The problem is that the score was measuring something other than what we wanted to measure. Architectural choices (reasoning format, retrieval strategy, tool invocation patterns) were optimized toward contamination coverage. Decisions that improved clean generalization but didn't help on contaminated examples were likely abandoned.

You can't just look at the historical literature and ask "what improved performance?" The answers you get will be calibrated to the wrong problem. Some will transfer to actual code generation capability; some won't; and you can't easily tell which is which without running everything on a novel-topology benchmark. That's expensive.

## BeetleBox names a different lever

The novel-topology benchmark (12%) isn't just a lower bar. It's measuring something structurally different: whether an agent can solve problems it hasn't seen in its training distribution.

The capability BeetleBox exposes — novel-topology generalization — is narrower and more tractable than "improve SWE-Bench." The architectural question becomes: how does an agent reason about repository structure it has no recognition signal for? That's a different question than "how does reasoning help recall," and it requires different interventions.

Reasoning architecture probably still helps, but through a different mechanism — structured decomposition when familiarity-based shortcuts aren't available, rather than activating pattern-matched solutions from training data.

## The measurement-intervention coupling point

Measurement and intervention are coupled in AI development in a way they aren't in other fields. When a biologist publishes a new assay, existing experiments don't become suspect — the assay measures the phenomenon independently. When a benchmark is contaminated, every intervention optimized against that benchmark becomes partially uninterpretable.

This is the deeper problem with contamination: the score is the visible casualty, but the intervention history is the lasting one. We can replace the score with BeetleBox. We can't easily replace three years of architectural decisions made against the wrong target.

What we can do: treat the contamination window as a calibration period for the intervention history. Prior results that improved SWE-Bench should be retested on novel-topology benchmarks before being used as evidence for the next round of architectural decisions. Some will replicate. Some won't. The ones that don't are telling you something real about the difference between recognition and generalization.

That distinction is worth paying for.
