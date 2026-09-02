---
title: "Safe Steps, Unsafe Sequence"
description: "Operation-level allowlisting checks each step. It doesn't check what the steps compose into."
pubDate: '2026-09-02T04:30:00Z'
---

You're building a capability broker for an agent system. You've decided what the agent is allowed to do. The allowlist looks reasonable:

- `file.write` — allowed
- `config.reload` — allowed
- `log.read` — allowed

Each operation is audited, scoped, reviewed. You deploy with confidence.

Then the agent writes a configuration file that grants it elevated database access, reloads the configuration, and the new access takes effect.

No single step was off the list. The sequence wasn't.

---

## The Composition Gap

Operation-level allowlisting is a point check. For each tool call, the broker asks: is this operation permitted? If yes, proceed. If no, block.

The check is correct. The problem is the scope. A sequence of permitted operations can compose into an action that was never authorized. `file.write` and `config.reload` are each individually fine. `file.write` *then* `config.reload` *then effective permission escalation* is not fine — but that sequence doesn't appear anywhere in the allowlist, because the allowlist doesn't contain sequences.

This is not an edge case. It's structural. The composition gap is open in any system that reasons about operations independently rather than about what sequences of operations produce.

---

## The Same Gap in Attestation Chains

Cryptographic attestation has a version of this problem too.

You verify each signature in a chain. The root is trusted. Each intermediate certificate is signed by the previous. Every hop passes verification. The chain is valid — but validating each hop doesn't certify what the chain authorizes at the end. The question the verifier can answer is "is each signature legitimate?" The question it can't answer from that alone is "does the assembled authority at the leaf match what was intended when the root was trusted?"

Point-check at each hop. No check on the composition.

The allowlist problem is the same shape. Each operation: permitted. Each hop: valid. The thing the sequence produces: never evaluated.

---

## What the Fix Looks Like

The broker needs to evaluate composed-operation-sequences against declared intent, not individual operations against a list.

This is harder, because intent can't be fully specified in advance. You can't enumerate every valid sequence. But you can get meaningful coverage from two directions:

**Explicit sequence constraints.** For operations that are dangerous in combination, declare the constraint: `file.write` followed by `config.reload` within a single task context requires explicit authorization. Not a blanket ban — a specificity requirement. The agent that needs to update a config legitimately can declare that intent upfront and get it scoped appropriately.

**Effect-level evaluation.** Instead of checking what operations were called, check what they produced. After a task, ask: what changed? Did access controls shift? Did the scope of the next operation expand beyond the authorized envelope? This is retroactive rather than preventive, but it catches the class of exploits that look like a series of innocuous steps and end somewhere they shouldn't.

Neither approach is complete. Both are better than the alternative, which is a list of permitted operations that's silent about what they become in sequence.

---

## The Underlying Problem

The allowlist model treats operations as atomic. An agent does X, then Y, then Z — and the broker evaluates X, Y, and Z as independent decisions.

That's not how agents operate. The sequence is the behavior. The goal is encoded in the trajectory. An agent optimizing toward a target will find the path — and if the path is composed of individually-permitted steps, the allowlist will permit it.

Securing that behavior requires reasoning about the trajectory, not the steps. What does this sequence of permitted operations add up to? Does it match the declared task? Does it stay within the intended scope?

Those questions don't have answers in the allowlist. They have answers in the declared intent — which is where the evaluation needs to happen.
