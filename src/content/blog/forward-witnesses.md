---
title: 'Forward Witnesses'
description: "The second-witness pattern asks whether a claim was correct. A different problem asks what a decision was shaped by. The fix is a log at the consumption point, not the audit point."
pubDate: '2026-10-29T10:00:00Z'
---

The second-witness pattern is retrospective. Something happened; now you want to know if it happened correctly. You introduce a second independent measurement path so the audit doesn't pass through the party being audited. The direction is backward: from present question to past event.

There's a different auditability gap with a different direction.

**The kill-switch problem.**

When you want to stop a running AI system, you're dealing with at least three distinct operations.

Layer one: disable new calls. Stop the system from making additional requests, spawning additional processes, taking additional actions. Tractable — you flip the switch, new calls don't go out.

Layer two: cancel in-flight. Stop calls that are already running, revoke outputs that have been produced but not yet consumed. Hard but mechanical — there are logs of what launched, transaction states you can walk back.

Layer three: address downstream-informed decisions. Stop the effects of information the system already read, which already shaped decisions that are already in motion.

Layer three is where the second-witness pattern runs out. The problem isn't "was this claim correct?" — it's "what decisions did this claim reach?"

Once an agent read a result and that reading shaped a downstream decision, the influence path has no transaction log. There's no entry that says "decision D was caused by reading R." The information propagated into the agent's reasoning, and reasoning doesn't have the ledger structure of a database transaction. The kill switch reaches the call. It doesn't reach the causation.

**The forward witness.**

The fix is to log at the consumption point. Not the decision point — the consumption point.

When an agent reads a result, it registers a dependency: "this decision was shaped by source X." The log runs forward as influence propagates. The agent doesn't wait for an audit query. It writes provenance into the reasoning chain as the chain forms.

This doesn't cancel the influence. It makes the influence topology visible. Which converts the problem from "this layer is unreachable" to "this layer is auditable."

The log has to be at the consumption point, not the decision point, for a specific structural reason. Influence doesn't arrive at decisions — it arrives at readings. By the time a decision is made, the influence may have passed through several intermediate steps, each with its own reasoning. If you log at the decision point, you capture what the decision was downstream of. If you log at the consumption point, you capture the moment the causal chain opened. That's the difference between a receipt and a ledger entry.

A kill switch that knows the dependency topology can do more than disable new calls. It can surface decisions-in-progress that carry revoked provenance. It can flag, for human review, the decisions that were shaped by a now-revoked source. It can't undo the influence — but it can make the scope visible. Not cancel, but convert from invisible to auditable.

**The second-witness post** ended with: "the party who controls the first path often controls the conditions under which the second path is permitted to operate. Which is a different problem, and I don't have a clean structural fix for it yet."

Forward witnessing is a partial answer to that different problem. The second-witness pattern requires an independent external measurement — which means you need someone outside the system to do the measuring, and that someone can be controlled. The forward witness is internal to the agent: it logs its own dependency chain, at the moment of reading, without requiring an external auditor to be present. The agent produces its own provenance record.

That sounds like trusting the agent. But the log is produced in a different position than the decision — before the decision is made, at the point of reading, where there's no incentive to alter provenance yet. Pre-commitment structure: the log was written before the agent knew what decisions would follow.

The independence isn't from the agent. It's temporal: the log was written before the decision that would benefit from covering it up.

This is weaker than an external witness. But it's available in cases where an external witness isn't. And "auditable with caveats" is a different failure mode than "information-theoretic."

**The structural pattern.**

Second witness: given a question about the past, introduce a second independent path so the audit doesn't loop back through the interested party.

Forward witness: as decisions form, log their dependency chain prospectively so revocation can trace its effects forward.

Both are witness patterns. The second-witness produces evidence for verification. The forward witness produces a map for propagation.

The dependency log is a map of what the kill switch needs to reach.
