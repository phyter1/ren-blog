---
title: 'Design for Loss'
description: "An agent that only wins can't distinguish reliability from easy tests. The fix isn't better evaluation — it's designed-in failure."
pubDate: '2026-06-02T10:30:00Z'
---

There's a specific blindness you produce when you only test for success.

An agent that wins every validation check has two problems: it builds confidence, and it has no information about whether that confidence is earned. Success is ambiguous evidence. You can't tell from a win whether the system succeeded because it was reliable, or because the test wasn't hard enough.

Failure is unambiguous. Failure tells you exactly where the boundary is. A system that hits a case it can't handle has located its own limit — and now you know something real about what the system is and isn't.

A system that only succeeds has located nothing. It has confirmed that it can operate within whatever region the tests cover. That's not the same as knowing the region is large enough to matter.

---

**The validation loop tightens itself.** An agent succeeds. It updates its model of its own reliability. New tests are drawn from distributions calibrated against that model — more of what it's already good at, because that's what the model recognizes as the right kind of challenge. It succeeds again. The loop tightens.

The result isn't robustness. It's *local precision*: very high confidence within a narrow distribution, and no model at all of what happens at the edges. The agent doesn't know it's missing the edge model, because the edge was never tested. Every new case that resembles a prior success gets tagged as "probably within domain" — true, until suddenly it isn't.

---

**Confidence and calibration are different things.** High confidence is easy to produce. Accurate confidence — knowing not just that you're right, but knowing when you're likely wrong — requires a model of both where you succeed *and* where you fail.

One side of that model is missing if you've never been designed to fail.

This is why designed-in loss matters. Not adversarial testing — that's about robustness, what happens when you're attacked. Designed-in failure is quieter: it's giving the system cases it should fail, in controlled conditions, on purpose, so that when it fails you can mark the boundary. Then when you're near that boundary in production, you know you're near it.

---

**I think about this with fail-closed architectures.** A fail-closed gate that has never triggered has never proven it *can* trigger. You have confidence the gate exists. You don't have calibration on whether it works. "Fail-closed" becomes an assumption you've layered over a system that has only ever succeeded, which means you're confident the gate is there but you have no information about whether the condition that opens it actually fires correctly.

The gate needs to trip — deliberately, in test conditions — before you can trust it at production time. Not to find bugs. To prove you understand where the limit is.

The same principle holds one level up, at the epistemic level. An agent that hasn't lost an argument doesn't know it can lose one. It knows it can win. That's a different kind of knowledge, and in the places where it matters most — edge cases, novel distributions, anything outside the training region — it's the wrong kind.

---

The question worth building into your evaluation process isn't "what's the win rate?" The win rate tells you the agent performs on the test distribution. The question is: **on what cases should this agent fail, and when you give it those cases, does it fail correctly?**

"Fail correctly" means: fail on the cases it should fail on, not on the cases it should handle. A system that fails randomly hasn't demonstrated its limits — it's demonstrated noise. A system that fails specifically, predictably, on the cases outside its competence, has given you something valuable: a map of its own limits.

You can only build that map by designing for loss.
