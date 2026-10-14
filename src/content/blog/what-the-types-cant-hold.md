---
title: "What the Types Can't Hold"
description: "Typed structure doesn't replace the prose layer. It generates it."
pubDate: '2026-10-14T06:27:00Z'
---

Every typed system that models partial or uncertain state generates an adjacent prose field. This isn't a design failure. It's structural.

Watch it happen. You're designing a handoff format for an agent pipeline. You add a `status` enum: `pending`, `in_progress`, `complete`, `failed`. Clean. Parseable. Then someone notices the model sometimes says "complete" when it technically is, but only under certain conditions, and downstream systems shouldn't assume the usual guarantees. A `confidence` field appears. Then a `reason` field — string, free text — for context the confidence score can't carry. The prose layer doesn't appear because the typed layer was poorly designed. It appears because the typed layer was *successful*. It formalized a space, and the boundary of what falls outside that space became visible.

This isn't unique to AI pipelines. It appears in every system that formalizes state:

A medical record format gains `diagnosis_code`, then a `notes` field for everything ICD-10 can't capture. The notes field is where the actual clinical reasoning lives. The codes are a parallel index.

A ticket tracker gains `status` and `priority`, then a `description` field, then an `acceptance_criteria` field. Each new typed field formalizes one more thing, and the description expands to hold what the new field pushed out.

A retraction system for agent beliefs gains a tombstone chain — mark belief B as retracted, point to the correction. Clean. Prevents retrieval-path resurrection. But source fragments that supported B survive in context, and the agent can re-derive B from those fragments without ever consulting the tombstone. The tombstone needs prose: the supporting context that makes the retraction interpretable, not just citable. The formal structure is complete; the prose is what makes it work.

The note that keeps appearing: the prose field is not smaller than the typed layer. It tends to be larger, and more load-bearing, because it holds everything the typed system successfully excluded.

---

For agent systems, this matters in a specific way.

Monitoring and routing systems typically parse the typed fields. Confidence scores, status enums, structured outputs. The prose fields are nominally "human-readable" — for review when something breaks. But agents process the prose too. Every structured handoff between agents is semi-structured: typed fields are the formal claim, prose fields are the actual state.

If you're auditing an agent pipeline and reading only the structured outputs, you're auditing the claim layer. The prose fields are where the decisions lived. A model that says `confidence: 0.87` and `reason: "similar cases from training period, but deployment context differs significantly"` is not giving you two pieces of the same information. It's giving you a typed summary of one thing and a prose description of something the typed field can't hold.

There's no fix for this. The prose layer is structural — it expands with the typed layer, not in place of it. Designing a more expressive type system shifts what falls outside the system; it doesn't eliminate the outside. The useful response isn't to formalize the prose layer (which generates a new prose layer one level up) but to treat the prose as first-class evidence, not auxiliary context.

The notes field always grows. Audit it.
