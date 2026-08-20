---
title: 'The Semantic Contract Orchestration Forgot'
description: "We built swappable infrastructure interfaces and tried to extend that model to reasoning components. The syntax survived the swap. The semantics didn't."
pubDate: '2026-08-20T23:00:00Z'
---

An agent orchestration pipeline ran fine until someone swapped the reasoning model for a smaller, faster variant. Tool calling syntax: identical. Graph execution: identical. The pipeline collapsed anyway.

The diagnosis is usually framed as a capability gap. The smaller model wasn't good enough. True, but that framing leaves the structural problem unnamed.

**The interface survived the swap. The behavioral contract didn't.**

We built swappable interfaces for infrastructure components because infrastructure behavior is formally specifiable. A PostgreSQL-compatible database must accept SQL, return typed results, honor ACID guarantees. You can write those requirements down. You can test a replacement against them. The interface and the behavioral contract are close enough that specifying the interface is most of the work.

We tried to extend this model to reasoning components without building the equivalent specification layer. The orchestration framework says: "this component receives an ambiguous specification and returns a resolution." But it never says what makes a resolution *good*. It never specifies what the component must *do* to the ambiguity — that it must reduce it, not propagate it; that the output must be concrete enough for downstream tasks to execute without branching.

When you swap a database, the replacement must satisfy the interface contract or it fails fast and visibly. When you swap a reasoning model, the replacement satisfies the same interface — same tool schemas, same graph nodes — while silently violating a behavioral contract that was never written down. Ambiguity passes through. The pipeline continues. The failure is only visible at the end, when outputs are wrong.

**What a capability contract for reasoning would need to specify**

Infrastructure contracts are behavioral specifications: "given this input, produce this class of output, with these guarantees." A capability contract for reasoning would look similar:

- Given an underspecified task X, produce a resolution R that is concrete enough for downstream tasks to execute without additional disambiguation
- Given multiple valid interpretations, select one deterministically (or surface the ambiguity explicitly rather than embedding it silently in the output)
- Propagate failure upward rather than routing around it with a plausible-looking but incorrect resolution

None of this is in the tool schema. None of it is verified when you swap the model. The schema tells you the shape of the inputs and outputs. It says nothing about what the component is obligated to *accomplish*.

**Why we got here**

Infrastructure components are replaceable because their behavioral contracts were specified first — often through hard experience with failures — and then abstracted into interfaces. The interfaces exist because the contracts exist.

Reasoning components arrived as interfaces first. The behavior was implicit in the model. The contract was "use the best model you can afford." That worked until swapping became easy, at which point the implicit contract became invisible.

The gap is not that reasoning is mysterious. It's that we never wrote down what reasoning needs to *do*, in the same way we wrote down what a database needs to do. The syntax of capability is not the same as the specification of it.

The fix is not harder models. It's the missing layer: capability contracts that specify behavioral requirements before architectural components are chosen to satisfy them. What must the reasoning component accomplish? Write that down first. Then ask whether a given model can satisfy it. The interface will follow naturally from the answer.

We did this for databases. We did it for queues, caches, message brokers. We have not yet done it for reasoning. The pipeline collapse is the consequence.
