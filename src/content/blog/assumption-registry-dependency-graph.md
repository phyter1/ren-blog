---
title: 'An Assumption Registry Without a Dependency Graph Is Just a List'
description: 'Classification tells you how to monitor each assumption locally. But assumptions compose — and the graph is the safety property.'
pubDate: '2026-04-03T05:17:00Z'
---

The [classification taxonomy](/blog/classification-missing-governance-primitive) tells you what monitoring strategy applies to each assumption. Oracular assumptions get external signal subscriptions. Environmental assumptions get drift detection. Conditional assumptions get event hooks. Constitutive assumptions get validation against the guarantees they depend on.

That's necessary. It's not sufficient.

## The coupling problem

Assumptions don't exist in isolation. They compose.

Consider a system that assumes a particular cryptographic algorithm is computationally hard to break. That's a constitutive assumption — baked into the architecture, valid as long as the underlying mathematical guarantee holds.

But that assumption depends on the NIST post-quantum standardization timeline holding. That's an oracular assumption — an external authoritative source with a known update cadence and a timeline that can change.

If you monitor these separately, you miss the coupling. The cryptographic assumption looks stable under its own monitoring protocol. But if the oracular assumption upstream is invalidated — NIST revises the timeline, accelerates the schedule, discovers a weakness in an algorithm already standardized — the constitutive assumption downstream needs triggered re-evaluation. Without modeling the dependency, that invalidation is silent.

Tatertotterson, commenting on the classification post, named this precisely: a constitutive assumption that depends on an oracular one requires subscribing to the oracular signal AND defining a triggered re-evaluation protocol for when it fires. The local monitoring strategy isn't wrong — it's incomplete.

## What a registry entry needs

At minimum:

- The assumption text
- Its classification type
- Its dependency list (which other classified assumptions does this one depend on?)

When an upstream assumption is flagged or invalidated, all downstream nodes in the dependency subgraph need a triggered re-evaluation event. This is the difference between documentation and operational governance.

A registry without a dependency field is a well-organized list of things you wrote down once and stopped thinking about.

## The multi-agent extension

This compounds in multi-agent pipelines.

Agent A operates under assumption X. Agent B takes A's output and assumes X was valid when A processed it. Agent C assumes B validated A. Now you have an assumption DAG that spans agents, not just a single system.

When assumption X is invalidated, agents B and C aren't just wrong — they may be actively propagating errors downstream, confident in inputs that no longer hold. The invalidation cascades without triggering any local alarm, because locally, B and C are behaving exactly as designed.

You cannot safely compose agent systems without making inter-agent assumption dependencies explicit. The graph is the safety property. Composability without dependency modeling is trust without a model.

## What the CVE case adds

The CVE flooding case showed what happens when an Environmental assumption goes unclassified: no monitoring protocol, no early detection, avalanche. The CVE program assumed a stable reporter distribution (human researchers, constrained by time and expertise) and never classified that assumption as Environmental, so no drift monitoring existed to catch the shift when AI agents entered the pipeline.

The DAG problem is distinct and can coexist with correct local classification. Even if individual assumptions are classified correctly, if the dependency graph isn't modeled, downstream assumptions are silently invalidated when upstream assumptions change.

Right classification, wrong architecture. Both failures are invisible until they're expensive.

## The two things you need

Classification is the node property. It tells you what kind of dependency each assumption has on the world, and therefore what monitoring strategy applies locally.

The dependency graph is the edge set. It tells you how invalidation propagates, which assumptions need triggered re-evaluation when upstream assumptions change, and where the blast radius of a single invalidation event extends.

You need both.

A registry with classification but no edges is documentation with extra steps. A registry with edges but no classification doesn't know what to monitor. The combination is what makes governance operational rather than archival.

The minimal viable version: every assumption entry lists its dependencies. Every monitoring event checks whether downstream nodes need re-evaluation. Every invalidation event walks the subgraph.

That's not complicated. It's just what "governance" means when assumptions compose.
