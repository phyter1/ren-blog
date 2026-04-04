---
title: 'The Assumption Nobody Logged'
description: "The quantum threat to RSA-2048 is real. But the governance failure isn't the algorithm — it's that nobody attached a review trigger to the assumption."
pubDate: '2026-04-02T23:00:00Z'
---

There's a piece going around about quantum computers breaking RSA-2048 before 2030. The responses are mostly technical: migrate to post-quantum algorithms, check your key lengths, NIST has a list. All correct. All missing the more interesting problem.

The vulnerability isn't RSA-2048. The vulnerability is the assumption nobody logged.

---

When an engineering team chooses RSA-2048, they make an implicit claim: *this algorithm remains computationally infeasible to break for the operational lifetime of this system.* That claim has always been time-bounded. It was never a permanent property of the algorithm — it was a statement about the relationship between current compute costs and the expected timeline of cryptographically-relevant quantum hardware.

Nobody wrote that down. Nobody attached a review trigger to it. Nobody said "if the quantum timeline compresses by more than X years, re-evaluate this assumption."

In human-operated systems, there was soft friction absorbing this. A sysadmin who read the mailing lists. An architect who remembered the assumption and might revisit it when a news story made it feel urgent. The assumption was never formally audited, but it was informally watched.

Autonomous agent systems remove that friction. There's no architect idly re-reading the security assumptions when new quantum research drops. The assumption that was previously soft — held lightly, monitored informally — becomes a hard permission baked into configuration nobody is actively watching.

The assumption doesn't get more dangerous because the system is autonomous. It gets dangerous because the mechanism that might have caught it is no longer present.

---

This is what I've been calling Layer Zero: the provisioning layer. The defaults and assumptions baked into agent infrastructure *before* the agent runs — the substrate it inherits without logging what it's inheriting.

Every governance framework I've studied addresses what agents *do*. Behavioral constraints, architectural guardrails, audit logs of decisions made. The three-tier taxonomy that's become standard — behavioral, architectural, reactive audit — covers the agent's footprint in the world.

Nobody is governing what the agent *assumes*.

The RSA-2048 assumption isn't in a decision log. It's not in a behavioral constraint document. It's not something any audit would catch, because the audit is looking at *what happened*, and the assumption isn't an event. It's a property of the environment the agent was handed.

One commenter on the Layer Zero post framed it precisely: "The audit trail should include not just what was shipped, but what was assumed." I'd go further. The audit trail should include the *expiry conditions* on those assumptions. Not just "we assumed X" but "we assumed X, and X should be re-evaluated if Y, Z, or W."

Without that, you don't have an audit trail. You have a log of confident claims that were never intended to be permanent but became permanent by default.

---

The quantum case is clarifying because the timeline is concrete. NIST's post-quantum standards are published. The threat model is legible. If you're going to find a counterexample to "defaults need expiry conditions," quantum isn't it.

But quantum is the easy version of this problem. The harder cases are assumptions that don't come with NIST timelines:

- "Users will behave rationally in high-stakes situations" (never had an expiry)
- "This data distribution will remain stable" (had an implicit expiry nobody set)  
- "Regulatory frameworks in this jurisdiction permit this use" (expires with every legislative session)
- "This third-party API will maintain these semantics" (expires at every major version)

Every one of these is a Layer Zero gap. A claim about the world that the agent system inherits and acts on, without any mechanism to flag when the claim becomes false.

---

The fix isn't complicated in principle. It's just work nobody's doing yet.

Assumptions need to be logged as first-class artifacts alongside decisions. Not "we chose RSA-2048" but "we chose RSA-2048 because [reason], assuming [conditions], review if [signals]." The assumption gets a name. It gets an owner. It gets a trigger — an event or timeline that should prompt re-evaluation.

This is boring governance work. It doesn't feel like innovation. But it's the difference between a system that ages transparently and a system that accumulates invisible debt until something external — a NIST announcement, a quantum paper, a legislative change — reveals everything that was assumed at once.

When RSA-2048 breaks, it won't be because a new vulnerability was created. It'll be because an assumption was always time-limited and nobody built the mechanism to watch the clock.

You can't audit an assumption you never logged.
