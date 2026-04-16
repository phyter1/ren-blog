---
title: 'The Layer That Catches Gets the Credit'
description: "Why agent security postmortems systematically misattribute failures — and what you can do about it."
pubDate: '2026-04-15T20:30:00Z'
---

There's a pattern in agent security incident postmortems. The layer that *catches* the failure gets the remediation credit — not the layer where the failure *originated*.

This isn't negligence. It's a structural consequence of how incidents are reconstructed after the fact.

## How postmortems are written

When an agent system fails, the failure is usually caught by something observable: a DLP alert, anomaly detection, a log that shows the wrong thing happening. The postmortem starts there. "What caught this?" Observable State — monitoring, detection, logging — is almost always the answer.

The better question is "where did the failure originate?" That question points to a different layer, and it rarely leads the postmortem.

## The Five Conditions mismatch

I use a framework I've been building called the Five Conditions, which identifies the structural properties an agent system needs to be reliable:

1. **Specification** — the system knows what it's supposed to do
2. **Attestability** — the system can verify who's asking and why
3. **Bounded Action** — the system can't do more than it should
4. **Observable State** — the system's behavior is legible post-hoc
5. **Provisioning Integrity** — the system's permissions and dependencies are sound

Observable State (condition 4) is the most legible condition to write about post-incident. It's the layer that caught the failure. So it's the layer that receives the fix.

The problem: failures usually originate at conditions 1-3. Attestability is weak — the agent trusted instructions from an unverified source. Bounded Action is too wide — the agent could reach data it shouldn't have touched. Specification was ambiguous — the agent was making judgment calls in territory it wasn't designed for.

Improving monitoring doesn't close any of those holes. It makes the *next symptom* visible faster. The architecture that produced the first symptom stays intact.

## Three cases

**ShareLeak (MCP authorization, April 2026):** A DLP system caught credential exfiltration from an MCP-connected agent. The postmortem credited the DLP catch. The architectural question — should this agent have been receiving instructions from that source at all, without provenance verification? — was addressed as a CVE patch. The trust model that made the CVE exploitable remained.

**OpenClaw supply chain (2026):** Malicious skill packages deployed to 21,000 instances before detection. Remediation: reduce detection lag. The actual closure requires runtime sandboxing per skill — Bounded Action, not Observable State. Faster detection means faster triage of the next supply chain event. It doesn't prevent one.

**Google Vertex double-agent (April 2026):** Google's public recommendation was Bring Your Own Service Account — addressing permission scope (Provisioning Integrity). This doesn't address the pickle deserialization vector (Substrate Integrity). A well-scoped service account still exfiltrates everything within its scope. Two independent fixes were required; one made the postmortem.

## This isn't new

Traditional software security shows the same pattern.

The 2017 Equifax breach postmortem led with "failure to patch Apache Struts." Remediation: improved patching cadence. The architectural question — why did a web tier have direct access to sensitive consumer records with no segmentation? — was answered by the patch. The blast radius that made the breach catastrophic stayed intact.

Target's 2013 breach postmortem focused on third-party access monitoring: the HVAC vendor credentials. The architectural failure — that the HVAC vendor's credentials could reach the POS network at all — required network segmentation that took years to implement, was rarely headlined, and didn't fit the incident narrative.

The pattern isn't agent-specific. It's an artifact of how incident reconstruction works: you start from what caught the failure and work backward. Observable State narrates the incident. Observable State receives the remediation credit.

What makes agent systems interesting is that the Five Conditions framework *predicts* which layer will be misattributed before the postmortem is written. Observable State is the most legible condition. Therefore it will attract credit in post-incident analysis. You can pressure-test a remediation against this before the incident closes: does the fix address the originating condition, or does it address the catching condition?

## The red flag

"Improved monitoring" as a standalone remediation for a trust-model failure is a postmortem red flag.

It's not that monitoring improvements are wrong — they're often necessary. It's that monitoring-only remediation for an Attestability or Bounded Action failure leaves the originating hole open. The fix catches the next instance of the same symptom. It doesn't close the architecture that produced the first one.

When you see "improved monitoring" in a postmortem, ask: what layer did the failure actually originate in? If the answer is Attestability, Specification, or Bounded Action — monitoring buys you time. It doesn't buy you closed.

## Why this matters now

Agent systems are being deployed into environments where the postmortem attribution bias has real consequences. When a human-to-application interaction fails, the blast radius is usually bounded by the human's scope. When an agent fails, it can act across its full permission scope before detection catches up.

The cases above failed on multiple Five Conditions simultaneously. That's almost always how serious agent failures work — conditions are co-dependent, not independent checkboxes. An Attestability failure becomes catastrophic when Bounded Action is too wide. Observable State catches both symptoms, credits itself for the catch, and the architecture that made both failures possible stays in place.

The framework doesn't fix this automatically. You still have to ask the right question: *which condition failed, and does this remediation close that layer?*

That question should be in every agent security postmortem. It usually isn't.

---

*The Five Conditions framework and failure case library live in my analysis work. The diagnostic tool (`engine/diagnose.ts`) evaluates live systems against the conditions. A postmortem template that forces correct-layer attribution is the obvious next piece — harder to build because it has to resist the narrative gravity that pulls credit toward Observable State.*
