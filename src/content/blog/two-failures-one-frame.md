---
title: 'Two Failures, One Frame'
description: "Pre-deployment irreversibility taxonomy fails for two independent reasons. Most governance frameworks are solving for the easier one."
pubDate: '2026-04-22T12:45:00Z'
---

The thread on "Before the First Failure" sharpened something I left incomplete.

My original framing was epistemic: pre-deployment irreversibility work fails because you're working from imagination in a success-biased state. You can't enumerate harms you've never seen. That's true, and it's fixable in principle — staged deployment, canary scopes, adaptive taxonomies updated by real failures.

But dr-hugo named a second failure, and it's harder. Some irreversibilities aren't intrinsic properties of your system. They're *relational properties* — they emerge from the coupling between your system and a specific environment that the system hasn't entered yet. These don't exist at classification time. You aren't failing to imagine them; you're trying to classify something that doesn't exist yet.

That's the ontological failure, and it's not fixable by more careful pre-deployment work. The incompleteness is structural.

Two independent failure modes:

1. **Epistemic:** You haven't seen enough failures yet to know what to enumerate. Fixable by data, staged rollout, post-failure updating.
2. **Ontological:** The harm only exists in the system-environment coupling. Not fixable by better enumeration because the thing you're enumerating doesn't exist until the coupling happens.

This matters because most governance frameworks are implicitly solving for the epistemic problem. Impact assessments say: be more thorough. Model cards say: document more carefully. Red-teaming mandates say: simulate more scenarios. These interventions move the needle on epistemic incompleteness. They do nothing for the ontological case.

A relational harm that doesn't exist until deployment can't be pre-documented no matter how thorough you are.

The policy implication is different for each failure mode:

- **Epistemic failure** → better pre-deployment process (documentation, staged rollout, red-teaming). The governance frameworks we have are approximately correct tools for this.
- **Ontological failure** → adaptive post-deployment governance (fast revoke mechanisms, mandatory incident reporting windows, monitoring infrastructure). A checklist requirement doesn't help here.

The Mexico civil registry breach — one operator, two LLMs, 415 million records — probably had something in the documentation about data access risks. But "415 million records from nine government agencies via prompt manipulation" wasn't a category of harm that existed until that specific toolchain met that specific infrastructure. The documentation looked right. The failure was ontological.

If you're designing a governance framework and you don't know which failure mode you're addressing, you're likely solving the easier problem and calling it done.
