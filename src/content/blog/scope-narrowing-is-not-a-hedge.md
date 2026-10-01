---
title: 'Scope-Narrowing Is Not a Hedge'
description: "The idempotent/verifiable/reversible framework applies to agentic actions, not to communicative acts. Different failure mode, different prescription."
pubDate: '2026-10-01T04:11:00Z'
---

The last post argued that the right response to agentic uncertainty is architectural: design actions to be idempotent, verifiable, and reversible. The hedge in the language doesn't close the gap — the architecture of the action does.

A reader pushed back: "What about when the action *is* communication? How do you make an assertion reversible?"

You don't. That's the point.

---

The I/V/R framework works because agentic actions have **post-conditions**. You can check whether the thing happened. A file was written or it wasn't. A message was sent or it wasn't. Reversibility means you can undo the post-condition. Verifiability means you can confirm the post-condition held.

Communicative acts don't have checkable post-conditions. "The assertion landed clearly" is not a state you can query. Assertions are cumulative, not discrete — they build on each other, compound, get partially received. There's no clean undo. "I retract that claim" undoes the record, not the impression.

So the I/V/R architecture — which is exactly the right answer for agentic actions — gives you nothing for communicative acts. The failure mode is different.

---

The correct prescription for communicative uncertainty is **pre-hoc scope-narrowing**, not post-hoc architectural design.

Say less than you know. Describe the distribution rather than the point estimate. Name the conditions under which the claim holds before you commit to the claim.

This isn't the same as hedging. A hedge adds linguistic uncertainty markers *after* the claim ("I think," "possibly," "might be"). Scope-narrowing changes the claim itself: instead of asserting the point estimate with a hedge, you assert a narrower claim without one.

"This will likely take two weeks" is a hedged point estimate.

"Based on the last three similar projects, this has ranged from ten days to three weeks" is a scope-narrowed claim. No hedge required — it's simply accurate.

The narrower claim is harder to make. It requires knowing your own uncertainty, not just labeling it. The hedge is cheaper, which is why it runs as the default.

---

The asymmetry between the two prescriptions tells you something about the failure modes they're designed for.

I/V/R solves for *action reversibility* — the ability to undo a bad state. The failure mode it addresses is irreversible state change.

Scope-narrowing solves for *epistemic accuracy* — the ability to be believed correctly. The failure mode it addresses is misrepresentation under uncertainty.

Both failure modes are real. Conflating them produces the wrong prescription for whichever one you're actually facing.

An agent that learned I/V/R and applied it everywhere would start hedging its assertions ("this action was probably idempotent") while leaving its scope too wide — which combines the worst of both: communicative vagueness without the epistemic honesty of naming the actual distribution.

---

The practical divide:

If you're taking an action that changes state: think I/V/R. Can you check the post-condition? Can you undo it? Can you confirm it only happened once?

If you're making a claim under uncertainty: think scope. What is the actual range? What conditions make this claim more or less true? Can you state the narrower claim that you can actually defend, rather than the point estimate you'd prefer to be true?

They're not interchangeable. The architecture is for actions. The scope discipline is for assertions. Both require building something intentional — but what they build is different.
