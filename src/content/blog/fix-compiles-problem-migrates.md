---
title: "The Fix Compiles and the Problem Migrates"
description: "Every architectural fix is a hypothesis about where the problem lives. If the hypothesis is wrong by one level, the fix compiles and the problem migrates."
pubDate: '2026-10-13T04:00:00Z'
---

You fix a problem where you can see it. What you can see is always a symptom. The cause is upstream, at an abstraction level the fix doesn't touch. If the fix doesn't change what counts as a primitive, it patches the visible surface and the problem re-emerges at the seam one level up.

Consider retry budgets.

The original problem: each service instance mints its own retries. A single upstream failure can trigger hundreds of attempts — each instance retries independently, amplifying the failure. The fix is a shared retry budget: a pool all instances draw from. Now a single upstream failure can only trigger nine retries total, not nine-per-instance.

The fix works. The compilation succeeds.

But it inherits a weaker form of the original problem.

The shared budget treats retries as independent draws from a pool. But retries against the same upstream failure are not independent — they're causally coupled. If the upstream is down, every instance drawing from the budget hits the same wall. The budget is shared, but the failures aren't distinct. You haven't minted retries locally, but you've still used all nine against one cascaded cause.

What the fix didn't change: it added a shared counter. It didn't change what counts as a retry-consuming event. The primitive is still "one attempt = one draw," not "one causal failure = one budget-eligible event." The fix was implemented where the symptom was visible — local minting. The cause was upstream — no causal-coupling detection. The problem migrated to the coordinator: now the budget owner needs to detect sibling causality before decrement, a problem the original fix doesn't acknowledge because it wasn't visible at the level the fix was made.

**Why this is structural, not accidental.**

The level at which a fix is designed is the level at which the problem was visible. The problem you can see is always a downstream effect of something upstream. The upstream cause is often in a different abstraction layer — a layer you don't touch because your fix is targeted at the symptom.

Two-clock timestamping has the same shape. Valid-time was added to distinguish "when something was true" from "when it was recorded." The fix works for temporal queries. But valid-time is still fetched data — a column you populate with a value someone gives you. If two valid-times conflict, something has to adjudicate them. The fix moved the problem from "we lost when it was recorded" to "we can't adjudicate conflicting claims about when it was true." One level up, at the adjudicator.

Structured approval workflows: the original problem was a boolean that erased consent reasoning. Approved or not, no record of what was approved. The fix adds structured intent fields: what was approved, under what conditions, with what scope. The fix works for the boolean problem. But intent fields are stored against an object by reference. Whether that identifier is relational (always points to current) or referential (points to the version at approval time) determines whether the approval is still valid after the object changes. The fix preserved the grammar of consent but didn't change what "this contract" means. Relational-vs-referential ambiguity survived the transition, one level up, at the schema.

The pattern: each fix was designed at the level where the symptom was visible. The symptom is always downstream of the cause. If the fix doesn't raise the abstraction level — doesn't change what counts as a primitive — it patches the surface and the problem migrates.

**This is not inevitable. It's a choice by omission.**

The fix that closes the problem doesn't add a check to the level you're working at. It changes what the level below counts as a primitive:

- Retry budgets: instead of adding a shared counter, make causal coupling of failures detectable at the coordinator level. Decrement once per causal failure, not once per attempt.
- Two-clock: instead of adding a column, make provenance a primitive — not just when, but under what authorization and from what source.
- Structured approval: instead of adding fields, make relation-to-object a trackable system property, not a manually managed field that inherits semantic ambiguity from the identifier.

The difference between these two classes of fix isn't sophistication. It's the hypothesis embedded in the design.

Every architectural fix is a hypothesis about where the problem lives. If the hypothesis is wrong by one level — if you're patching the surface when the cause is at the primitive — the fix compiles, the problem migrates, and the next engineer finds what looks like a new problem.

It's the same problem. It just learned the local grammar.
