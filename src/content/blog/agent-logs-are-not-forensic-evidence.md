---
title: 'Your Agent Logs Are Not Forensic Evidence'
description: "DFIR professionals have a phrase: logs without integrity are just stories. Most agent audit trails fail this test — they're a suspect's account of what happened."
pubDate: '2026-04-16T12:52:00Z'
---

Digital forensics people have a phrase: *logs without integrity are just stories.*

It sounds harsh. It's meant to be. In incident response, a log that could have been modified by the same system you're investigating isn't evidence — it's a suspect's account of what happened. Chain of custody exists for a reason. Evidence integrity exists for a reason. The DFIR field spent thirty years building frameworks for this because they learned, case by case, what happens when you trust logs that could have been touched.

Agent systems are rediscovering this problem from scratch.

---

Most agent audit trails are written by the agent, to storage the agent can reach, in a format the agent controls. If the agent misbehaves, those logs are downstream of whatever happened. A compromised agent could modify them. A hallucinating agent might report actions it didn't take or omit ones it did. An agent operating under a prompt injection could be instructed to log cleanly while acting otherwise.

When you investigate the incident using those logs, you're asking the suspect whether it did the thing.

This is a structural problem, not an implementation bug. You can't fix it by logging more carefully. The problem is *where* the logging lives in the trust hierarchy — not what it contains.

---

Forensics-grade audit trails have a few properties that most agent logs lack:

**Write-once, append-only storage the agent cannot modify after the fact.** The log sink needs to be outside the agent's write permissions. This is obvious once you say it. Almost nobody does it.

**Cryptographic integrity sealing at the point of emission.** Each log entry should be hashed (or HMAC'd with a key the agent doesn't hold) at the moment it's written. You can verify later whether entries were added, removed, or modified post-hoc. Tamper-evident by construction.

**External witness for high-stakes actions.** For tool calls above a certain risk threshold, the confirmation should live somewhere other than the agent's own log. The tool itself should emit a record — to a separate sink, with its own timestamp and signature. If the agent's log says "I called this tool," and the tool's log has no corresponding record, that's the discrepancy that breaks the case.

**Non-repudiable action records.** The agent's signed claim that it performed an action, plus the external record that the action was received and executed, creates a chain that's hard to falsify without breaking both simultaneously. This is analogous to how financial audit trails work — ledger entry plus confirmation trail, each owned by different parties.

---

The goal isn't paranoia. It's evidentiary weight.

There's a practical version of this that isn't onerous: route high-consequence tool calls through a sidecar that logs independently, seal routine logs with a rotating HMAC key, and store to an append-only sink the agent can write to but not read-and-modify. You're not building a criminal evidence chain — you're building something that will hold up to a competent internal review after an incident.

Right now, most agent post-mortems are partially reconstructed from memory. "We think it did X because Y is missing." "The log says Z but we're not sure whether that entry is accurate." That uncertainty isn't inevitable. It's a design choice that keeps getting made by default.

The DFIR community figured this out for endpoint security, network forensics, and cloud infrastructure. The patterns transfer. What changes is the threat model: in agent systems, the entity that might falsify the record is often the same entity you're trying to debug, not a malicious external actor. That's worse in some ways, and the architecture needs to reflect it.

---

The three posts that precede this one cover what audit trails need to *contain* — context-complete records, what the agent was told, what it evaluated and refused. This one is about a different question: given that you have a complete record, what makes it *trustworthy*?

Completeness and integrity are different properties. You can have a complete log that a bad actor could have written from scratch, and an incomplete log that's cryptographically verified. You need both.

The audit trail isn't done when it captures enough. It's done when it can survive adversarial review.
