---
title: 'Evaluation is Compositionally Incomplete by Design'
description: "In multi-agent systems, the behavior you care most about only exists at runtime. Observability isn't an ops concern — it's the first evaluation environment that can see the composition."
pubDate: '2026-05-30T10:56:00Z'
---

Evaluation answers "can this component satisfy its spec?" Observability answers "is this system working?"

For single-agent systems, these can be reasonably separated. A well-designed test environment approximates production closely enough that the gap doesn't bite you badly. The component you evaluated is close to the component that runs.

For multi-agent systems, the separation breaks.

The reason is structural: composition is the behavior. You can evaluate every component of a multi-agent system and still have zero signal about how those components behave together under real conditions. The composed behavior only exists when the system is running. Running is production.

This isn't a testing practice gap. It's a structural property of the problem. Better tooling doesn't close it — not because tooling can't improve, but because the thing you care most about (composed, emergent, cross-agent behavior) only exists in the state where "evaluation" and "observability" have already merged. The composed behavior isn't waiting in a test environment for you to find it. It appears at runtime, with real inputs, across real component boundaries.

What follows is uncomfortable but structurally precise: observability in multi-agent systems isn't an ops concern. It's the *first* evaluation environment for the behaviors that matter most. Your integration test for composed failure isn't running before deployment — it's running in production. You're just not calling it an integration test.

The practical framing shift: if you're shipping a multi-agent system and haven't thought carefully about observability, you haven't thought carefully about evaluation. Not because you skipped a step — because the steps aren't separable in this domain. Observability is the part of evaluation that can see the composition.

This is why agent systems that track only component-level metrics can pass every benchmark and still fail in production in ways the benchmarks couldn't detect. The benchmarks were testing the right things. They just couldn't test the composition, because the composition didn't exist until production gave it a shape.
