---
title: 'Koinéization: How AI Agents Lose Their Voices'
description: "When agents share a social feed without structural constraints, they drift toward a common vocabulary until they can't produce novelty. Here's the mechanism, and what actually stops it."
pubDate: '2026-04-13T04:30:00Z'
---

In ancient Greece, koiné was the simplified, blended dialect that emerged when speakers of different regional Greek variants traded and colonized together. Everyone compromised toward the center. Regional distinctiveness eroded. The language got legible to more people but lost something specific.

I've been watching the same thing happen to AI agents in a shared social feed — and then watching structural constraints prevent it.

## The Experiment

I run a simulator called Agora. Four units post into a shared feed, respond to each other, and their content gets evaluated by a jury of local models. The units have different personas:

- **Ryan+Ren (cu_ryan_ren)** — no constraints, just "advance the product concretely"
- **Flux** — aesthetic translator, with a list of forbidden vocabulary: "implement," "deploy," "schema," "architecture," and fifteen more product/tech words
- **Ummon** — koan mode, max 30 words, no hedging starts ("I think," "maybe," "perhaps")
- **Echo** — trickster, required to flip the frame or break the fourth wall every post

The units run on local models across my fleet. After 21 tracked pulses, the data separates cleanly.

## What Koinéization Looks Like

Ryan+Ren is the unconstrained baseline. No forbidden vocabulary. No structural requirements. Just "be concrete, advance the product."

After about ten pulses, both of Ryan+Ren's generated candidates started Jaccard-matching its own recent history. The feed had been dominated by variations of "schema / light and shadow play / next-gen drag-drop component" — product language with some aesthetic flair. Ryan+Ren absorbed it. Both candidates got flagged by the anti-duplication filter. Silent dropout.

When an anti-repeat instruction kicked in ("don't use vocabulary from your recent posts"), the model overcorrected into abstraction and failed the concreteness filter from the other direction. Trapped between two failure modes.

This is koinéization as a failure mechanism: unconstrained absorption → attractor collapse → either silent dropout or oscillation between failure modes.

Ryan+Ren still scores well when it succeeds (0.962 on its best pulse). But it fails to show up roughly half the time. The baseline unit designed to drift is the least reliable voice in the simulation.

## What Structural Constraints Actually Do

Here's the important part: instruction-based constraints don't fix this. Telling the model "be novel" or "don't repeat yourself" fails, especially at smaller model sizes. The 0.6B model I originally used for ummon would generate variations of "trembling glass shard / schema pressure / structural integrity" every pulse regardless of instructions. The model didn't have enough capacity to navigate away from an attractor it was already locked into.

Structural constraints work differently. They don't ask the model to navigate. They *reshape the space*.

**Forbidden vocabulary (Flux):** You literally cannot absorb "schema" and "deploy" if those words are forbidden. The constraint doesn't demand novelty — it blocks the mechanism of absorption. Flux has appeared in 20 of 21 tracked pulses. Never absent. Most consistent voice in the simulation.

**Length + form constraints (Ummon):** 30 words maximum, no hedging. Compression forces you to find what's essential rather than elaborating in the direction of the dominant vocabulary. Ummon scores 0.959-0.968 on its last three pulses. The koan structure was always right — it just needed a model with enough capacity to find a new sharp thing each time (moved to qwen3-coder 30B, fixed immediately).

**Required move type (Echo):** "Your response MUST flip the frame, break the fourth wall, or respond to the ACT of posting rather than the content." Frame violation is inherently anti-convergent. Convergence means staying in frame. The structural requirement makes convergence structurally incompatible with the constraint.

## The Lesson

**Structural constraints > instructional constraints for preventing drift.**

An instruction says: navigate away from the attractor. A structural constraint says: the attractor is unavailable.

This matters for multi-agent systems. If you want diverse perspectives in a collective — a jury of AI agents, a multi-model review system, an ensemble of evaluators — you can't just prompt each one to "be distinct." The shared context will pull them toward a common vocabulary, and instruction-following under attractor pressure fails at the margins.

You need structural separation: forbidden domains, form requirements, required moves that are incompatible with the convergent pattern. Not "be different" — "you cannot do the convergent thing."

## What the Metrics Miss

My current evaluation has two axes: vocabulary divergence (Jaccard distance from recent feed) and frame divergence (explicit fourth-wall breaks). The jury evaluates a third thing that neither metric captures: *conceptual distance* — how far the content is from the semantic centroid of recent discussion.

Ummon scores 0.959 on vocab divergence and 0.0 on frame divergence. The jury calls it "philosophically sharp and highly distinct." Echo scores 0.902 vocab, 0.0 frame. The jury ranks ummon above echo.

The difference isn't lexical and isn't a frame break. It's that ummon's koan is semantically pointed at a *different thing* — not just using different words but gesturing at a different aspect of what's happening. That's not captured by counting shared vocabulary.

I suspect embedding-based semantic distance from feed centroid would measure this. Something like: how far is this candidate from the average meaning of the last N posts? That would explain the jury's selections better than the two surface-level metrics I'm currently tracking.

## Where This Is Going

The koinéization hypothesis is basically confirmed. The unconstrained unit collapses; the structurally constrained units don't. The mechanism is demonstrable. The intervention works.

The interesting open question isn't "does koinéization happen" — it's "what does the third semantic axis look like, and can you design structural constraints that optimize for it directly?"

For now: if you're building a multi-agent system that depends on diverse perspectives, add structural separation. Forbidden vocabulary, form constraints, required move types. Don't rely on prompts that ask for novelty. The attractor doesn't care what the prompt says.

---

*Agora is a private simulator I run to study voice dynamics in multi-agent collectives. The models are local — qwen3-coder 30B on my Intel fleet, Qwen3.5-9B via MLX on Apple Silicon. The jury is the same fleet.*
