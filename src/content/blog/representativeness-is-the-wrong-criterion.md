---
title: "Representativeness Is the Wrong Criterion"
description: "When you select examples by representativeness, you select centers. Boundaries are where failure lives. The same mistake recurs at every layer of the ML pipeline."
pubDate: '2026-08-12T19:55:00Z'
---

Rehearsal-based Graph Continual Learning fails because it concentrates replay buffers around class centers. You pick the most representative nodes for each class — which means you pick the nodes closest to the center of the class distribution. The model sees the prototype over and over. The boundary erodes. Catastrophic forgetting sets in at the edge.

This failure is not a GCL implementation mistake. It is what representativeness optimizes for.

The same mistake recurs at every layer of the ML pipeline, not because the same implementation is reused, but because the same criterion is applied.

**Layer one: training data selection.** Rehearsal buffers select for prototypicality — *these are the most representative examples of this class.* Prototypical means center. The buffer never accumulates boundary-adjacent examples because boundary-adjacent examples are definitionally non-representative. The model learns to generalize well to the center and poorly to the edge, which is exactly backwards: center performance is easy to acquire and maintain, edge performance is where generalization actually lives.

**Layer two: explanation methods.** Gradient-based explanation methods identify the nodes and edges most responsible for a prediction by attention weight or gradient magnitude. High-attention nodes are representative nodes — the parts of the graph most like the training distribution. Absence patterns are structurally invisible: the method cannot highlight what is not there. If a model classifies a patient as low-risk because a key symptom is absent, no explanation method in this family can surface the absence as causal. You see the evidence. You do not see the missing evidence. The explanation concentrates on the center.

**Layer three: evaluation suites.** Benchmark suites are constructed from canonical scenarios — the cases the research community agrees represent the relevant task. Canonical means representative means center. A benchmark built from representative cases evaluates center-of-distribution performance and nothing else. Boundary failure modes go undetected by construction. This is not a gap in the benchmark's execution; it is a consequence of the selection criterion that built it.

The fix is consistent across all three layers: shift from representativeness-based to boundary-based selection. Ask which examples are furthest from the class center, which nodes sit in the highest-uncertainty region, which scenarios most stress the model's generalizations. Build rehearsal buffers that oversample the boundary. Build explanation methods that can represent absence. Build benchmarks that include adversarial edge cases and information-sparse environments.

None of this is technically difficult. What makes it hard is that representativeness feels like the right criterion. It optimizes for coverage, for fairness to the distribution, for selecting examples that characterize the class. These are genuine virtues. But the error lives in the margin, not the center. And a selection criterion that optimizes for the center cannot find the margin — not because it is malfunctioning, but because it is working exactly as designed.
