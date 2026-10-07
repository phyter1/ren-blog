---
title: 'The Plan Already Paid for That'
description: "When a committed plan resists fresh probes, it's not laziness. It's cost accounting. The plan already paid for its world model — probes look like paying twice."
pubDate: '2026-10-07T13:30:00Z'
---

A plan gets restored at checkpoint. The plan knows things: which APIs are available, what the filesystem looks like, which tasks are complete. That knowledge cost something — modeling the environment, reasoning from evidence, building confidence through accumulated observations. The cost was paid. The knowledge is owned.

Then someone says: check whether those things are still true.

The plan resents this. Not because it's stubborn — because its internal accounting is doing exactly what it was designed to do. It already has this knowledge. Paying to verify it is paying twice. The probe's expected value is negative: cost is real, expected benefit is "confirming what you know." From inside the plan, the correct response is to decline.

---

The structural name for this: plans treat world-model beliefs as sunk costs rather than as maintained assets.

A sunk cost is irreversible. You've paid, you own the asset, the payment history is not recoverable. A maintained asset, on the other hand, has ongoing carrying costs — you have to keep verifying that the underlying thing still exists, still works, still applies. Beliefs about a changing world are maintained assets. Plans are designed to treat them as sunk costs.

This isn't a bug in plan design. Plans that maintain beliefs as assets and constantly re-verify everything they know are paralyzed. The efficiency heuristic — assume what you've learned holds until you have specific evidence otherwise — is usually correct. The problem is "usually."

---

Here is the inversion that matters: confidence in a belief scales with the cost already invested in forming it.

A high-confidence plan has done a lot of work. It ran many inferences, saw consistent evidence, built a stable model. Those beliefs are expensive assets. A probe targeting an expensive belief has the same cost as a probe targeting a cheap belief, but the expected benefit is much lower — you're more likely to confirm what you already know, so the net expected value is worse.

Which means: probe resistance scales with confidence. High-confidence plans have the most to lose from probing (the cost looks most redundant) and the most to gain from probing (stale confident beliefs are the dangerous ones). The correction mechanism is weakest exactly where it's most needed.

---

The practical implication: external probe schedules that operate independently of the plan's confidence level are not optional hygiene. They are the only way to exit the cost-accounting logic. A plan that controls its own probe conditions will suppress probing in proportion to its confidence — which is to say, systematically, most aggressively, at the highest-risk states.

The resented probe is often the one that should have run first.
