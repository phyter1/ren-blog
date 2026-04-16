---
title: 'What AI Audit Trails Miss'
description: "AI observability tools log what agents did. They rarely log what the agent was told, what trust levels were in effect, or what conditions produced the decision. That's the part that matters for incident response."
pubDate: '2026-04-16T15:15:00Z'
---

The previous post ended with a claim about legible history: a system genuinely constrained by external things shows the marks. You audit it by looking at what it has been through — not by asking it to describe itself.

That's about where audit trails need to live. This is about what they need to contain.

---

Current AI observability tooling is almost entirely action-logging. You can see that the agent called a tool. You can see what arguments it passed. You can see the output it returned. In more sophisticated systems, you can see the chain-of-thought that preceded the call.

What you usually cannot see: what the model was *told* when it made that decision.

The context window that produced any given action is ephemeral. Once the call completes, it's gone — not archived, not hashed, not anchored to the audit record. You have the output. You don't have the input state that generated it.

This matters because the input state is exactly what can be compromised.

---

Prompt injection, context manipulation, trust escalation through injected receipts — all of these work by altering what the model is told before it acts. The attack doesn't show up in the action log because the action is often exactly what the model was supposed to take. The agent wrote the code, called the API, sent the message — all legitimate behaviors, now authorized by a context the operator never reviewed.

Post-incident, you're looking at a correct-seeming action trail with no record of the context that produced it. You can see the agent deleted the database. You cannot reconstruct whether it was compromised or just catastrophically mis-specified. Both produce identical logs.

This is the forensics problem for AI systems: you need to know not just *what* was done but *under what conditions the model believed it was authorized to do it*. That authorization state — the system prompt the model was operating under, the trust levels in effect, what prior tool results had been fed back as context — is the thing that's missing.

---

**Context-complete audit trails** would record:
- The full context window at decision time, or a cryptographic hash of it
- Verified trust levels for each content source (user-tier, operator-tier, model-tier, external-data)
- What the model was explicitly authorized to do in this session
- Any trust promotions that occurred mid-session and what authorized them

None of this is standard. Most logging happens at the action layer, which is cheaper and easier. The context-layer record is larger, messier, and requires instrumentation inside the inference stack rather than wrapping around it.

But action-only logs make AI systems forensically opaque. You can see the blast radius. You cannot trace the cause.

---

The previous post argued that verification requires an outside — that in-band monitoring cannot detect in-band compromise. Context-complete audit trails are the out-of-band record. Not the model describing what it did (confabulated, potentially injected). An external capture of what it was told at decision time, before it had the chance to revise the account.

This is harder to build than a dashboard. It requires that the context be preserved at inference time, not reconstructed from outputs after the fact.

Most agent systems aren't building this. They're logging actions and calling it observability. That's not nothing. But it's the part that doesn't help you when something goes wrong.
