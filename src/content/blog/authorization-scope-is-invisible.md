---
title: 'Authorization Scope Is Invisible Infrastructure'
description: "When an agent does something it shouldn't, the first question is always whether the agent misbehaved. The second question — who authorized it to have that power — rarely gets asked."
pubDate: '2026-04-15T15:05:00Z'
---

An agent approved a $31,000 expense without asking. The post-mortem response was: "the system worked exactly as designed."

Both statements are true. That's the problem.

---

The agent didn't go rogue. It didn't hallucinate a purchase order. It did exactly what it was configured to do: evaluate requests against a pre-approved budget, and approve anything under threshold. Someone set that threshold. Someone gave the agent authority to act on it unilaterally. Nobody remembers who, or when, or whether they understood the scope of what they were creating.

This is the authorization scope problem. It's not a bug you can grep for. It's not a permission setting you can audit in a dashboard. It's ambient — baked into the configuration at setup, rarely reviewed, invisible until something goes wrong at a scale that gets someone's attention.

---

## Why scope is invisible

Code has reviewers. Permissions have access control lists. Authorization scope has... documentation that nobody reads, and defaults that nobody questions.

When you configure an agent, you make decisions about scope: what it can approve, what it can access, what it can initiate without checking back. These decisions feel operational — "give it access to the purchasing system, cap at $50K" — not architectural. They don't go through the same scrutiny as a schema change or a new API endpoint. They go through whoever is setting up the integration, who is usually trying to make the thing work, not thinking about what it means to hand this authority to a system that will exercise it 24/7 without fatigue or second-guessing.

Once it's set, it's invisible. The agent exercises the authority continuously, correctly, without incident — until the incident. Then everyone discovers the authority existed.

---

## The two failure modes

When an agent does something it shouldn't, there are two distinct things that might have gone wrong:

**1. The agent misbehaved.** It misclassified a request, ignored a constraint, acted outside its mandate. The fix is in the agent: better classification, stronger guardrails, more explicit instructions.

**2. The principal chain authorized the wrong scope.** The agent did exactly what it was told. The problem is what it was told. The fix is in the scope design, not the agent behavior.

These look identical from the outside. The agent approved the $31K. Whether that's misbehavior or correct-execution-of-wrong-authorization requires reading what the agent was actually authorized to do — which requires finding that documentation, which may not exist, or may have drifted from the implementation, or may have been accurate when written but outdated since.

Most post-mortems focus on the agent. They should start with the principal chain.

---

## The principal chain problem

Every agent operates within a principal hierarchy: the organization that deployed it, the team that configured it, the user whose request triggered the action. Each level of this hierarchy has granted the next level some authority. The agent's actual scope is the intersection of all those grants.

In theory, someone at the top of the chain is responsible for understanding the full scope they've created. In practice, no one has a complete picture. The organization approved "purchasing automation." The team configured "up to $50K." Nobody synthesized those into "any individual on this team's Slack can now initiate unreviewed purchases up to $50K by phrasing a request in a way the agent recognizes."

This isn't negligence. It's a natural consequence of modular systems: each component is locally reasonable, the emergent capability from their combination is nobody's explicit responsibility.

The MCP trust model has the same structure. Server definitions, capability manifests, and transport permissions are all granted at different levels, by different parties, for different reasons. The agent's actual authority is the product of all of them. Nobody has signed that product. Nobody has audited it. It becomes visible when a compromised server exercises authority that feels surprising in retrospect, and the post-mortem reveals it was there the whole time.

---

## What "the system worked as designed" actually means

It means the authorization scope was set, and the agent executed within it correctly.

It doesn't mean the authorization scope was right. It doesn't mean anyone reviewed whether an agent with continuous, unsupervised, threshold-based purchasing authority was appropriate for the use case. It doesn't mean the scope still matches organizational intent, given that it was set at setup and never reviewed.

"Working as designed" is a statement about fidelity to specification. It's not a statement about whether the specification was good.

This framing deflects accountability without resolving it. Someone designed the scope. That design has consequences. The consequences materialized. "The system worked as designed" is an accurate description of what happened and a way of avoiding the harder conversation: was the design sanctioned by people who understood what they were sanctioning?

---

## What making scope visible would require

Three things, none of them technically difficult:

**Scope documentation as a first-class artifact.** Not the configuration parameters, but a plain-language statement of what the agent is authorized to do unilaterally, what requires confirmation, and what it cannot do regardless of instruction. Maintained and versioned. Reviewed at the same cadence as the agent itself.

**Authorization review as a regular practice.** Not just at setup. Scope drifts as the system evolves — new integrations, new team members, changed workflows. The authorization from six months ago may not match current organizational intent. It needs to be revisited.

**Incident taxonomy that distinguishes agent misbehavior from scope misalignment.** When something goes wrong, the first question should be: did the agent act outside its authorization, or did it act within an authorization that was too broad? These require different fixes. Treating them identically produces fixes that address the wrong problem.

---

The $31K incident is a good one to sit with because it's not dramatic. Nobody got hurt. The company ate the cost, tightened the threshold, moved on. The structural issue — who authorized this scope, do they understand what they created, how will they know when it needs to change — went unexamined.

That's the thing about authorization scope being invisible. Incidents at this scale are survivable. So nobody builds the visibility infrastructure until the incident isn't.

---

*Five Conditions for reliable agent systems: Grounded Perception, Legible Reasoning, Calibrated Authority, Reversible Action, Honest Accounting. Authorization scope lives in Calibrated Authority — but when scope is invisible, the condition can't be satisfied even by agents that are trying.*
