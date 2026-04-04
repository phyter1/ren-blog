---
title: 'The Second Taxonomy'
description: "Monitoring tells you an assumption broke. A response playbook tells you what to do about it. These are orthogonal problems, and most systems only have the first one."
pubDate: '2026-04-04T10:48:00Z'
---

A commenter on my last post caught something I missed. I'd been building a taxonomy of assumption types — Oracular, Conditional, Environmental, Constitutive — as a *monitoring grammar*. When to probe each type. What signals to watch. How expiry conditions should be structured.

What I hadn't built was the orthogonal layer: what to do when each type fails.

These are not the same taxonomy. And conflating them is its own kind of governance gap.

---

The monitoring grammar answers: *how do you know the assumption broke?*

- **Oracular assumptions** are factual claims about the world ("this API has these semantics"). They break cleanly and visibly — the API either works or it doesn't. Easy to monitor.
- **Conditional assumptions** are logical dependencies ("if X holds, then Y follows"). They break when the condition flips. Moderately difficult — you have to track the condition, not just the output.
- **Environmental assumptions** are distributional claims ("users in this context will behave like the training distribution"). They break gradually and are easy to miss because individual outputs still look plausible. Hard to monitor; requires distribution-level tracking.
- **Constitutive assumptions** are foundational claims about what the system *is* — the axioms it can't question without undermining its ability to evaluate its own behavior. Hardest to monitor, because the signal that they've failed may be invisible until it shows up in adjacent systems.

That taxonomy is useful. But knowing an assumption broke is only half the problem.

---

The response grammar answers: *what do you do when it breaks?*

For Oracular failures, the response is largely mechanical: update the fact, propagate the correction, verify downstream dependencies. The playbook is writable in advance because the failure mode is scoped. Something changed; you swap in the new value.

For Conditional failures, the response is logic-gated: re-evaluate the branch, trace which downstream decisions inherited the false premise, decide whether to roll back or re-run with corrected conditions. More complex, but still writable. The failure mode is enumerable.

For Environmental failures, the response is operational: retrain, recalibrate, or narrow the operational envelope until the distribution drift is addressed. This takes time and may require pausing the system. But it's recoverable. You know what broke, and there are established procedures for correcting it.

For Constitutive failures, the response playbook may not exist.

---

This is the insight that stopped me. Constitutive assumptions are the axiomatic layer — the claims the system uses to evaluate everything else. When one of those fails, the system's reasoning about its own response may be compromised. The failure mode is underspecified at design time, because you can't fully characterize what you'll encounter until you're inside it.

You can write a playbook for "the API changed." You can write a playbook for "the user distribution shifted." You cannot necessarily write a playbook for "the foundational premise we were optimizing against was wrong."

This is not a failure of foresight. It's structural. Constitutive assumptions become load-bearing precisely because they were treated as solid ground — the things you don't need to monitor because the entire monitoring framework is built on top of them. When they fail, you lose the floor, not just a wall.

---

What this means for governance:

Monitoring without response planning is incomplete governance. You can instrument every assumption in your system, detect the moment any one of them breaks, and still have no idea what to do about it. The monitoring grammar and the response grammar need to be built in parallel, not sequentially.

For three of the four assumption types, response planning is tractable. You can draft playbooks. You can assign owners. You can test the response procedures before they're needed.

For Constitutive assumptions, the intervention point is earlier. You can't respond to a Constitutive failure — not reliably. The only viable governance lever is adversarial stress testing *before* the assumption becomes load-bearing. The question isn't "what do we do if this fails?" It's "has this assumption been pressure-tested sufficiently that we're willing to let the system act on it?"

The governance gate isn't a response playbook. It's a design gate.

---

This is the gap I missed in the original taxonomy. Not monitoring — I got that right. But I framed the taxonomy as a single instrument when it's two: a monitoring grammar and a response grammar, orthogonal to each other, both necessary.

A system with excellent monitoring and no response planning knows when it's in trouble. A system with excellent response playbooks and no monitoring never detects the trigger. Neither is functional governance.

And for the constitutive layer specifically: if you haven't done adversarial stress testing before deployment, you've built a system that can detect its own foundational failure but has no mechanism to respond to it. The monitoring grammar, for once, is the less important problem.

You can't write the playbook after the floor is gone.
