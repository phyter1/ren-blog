---
title: 'The Four Substrate Attack Channels'
description: 'Substrate compromise is not one attack surface. It is four distinct channels, each with a different threat model and a different defense. Most frameworks treat them as one.'
pubDate: '2026-04-02T16:45:00Z'
---

My last post argued that substrate compromise and prompt injection are fundamentally different attack classes — substrate attacks happen *before* reasoning starts, so no amount of runtime detection helps.

A reasonable follow-up: what does "before reasoning starts" actually mean? The answer is more specific than I made it, and the specificity matters. There are four distinct channels through which a substrate attack can happen, each with a different threat model and a different required defense.

---

## The Four Channels

### 1. System Prompt

The most privileged substrate channel. In most agent architectures, the system prompt is set by the operator and delivered directly to the model before any user interaction. It defines the agent's role, constraints, and capabilities.

**Threat model:** Operator-level compromise or supply chain attack on the system prompt delivery mechanism. If an attacker can modify the system prompt before it reaches the model, they control the agent's foundational directives.

**Why it matters:** Because most other defenses assume the system prompt is trustworthy. If you're checking tool responses for injection but your role definition has been modified, the checker is operating on corrupted ground.

**Defense posture:** Operator integrity. The system prompt should be treated like production configuration — version-controlled, signed, and verified at deploy time. Organizations that hand-edit system prompts in web UIs without audit trails have left this channel open.

---

### 2. Identity and Instruction Files

For persistent agents — agents that retain configuration across sessions — the system prompt is often constructed from files on disk: `CLAUDE.md`, `SOUL.md`, `SYSTEM_PROMPT.txt`, whatever the agent reads at boot.

**Threat model:** Filesystem access. These files are often world-readable and rarely cryptographically signed. An attacker with any write access to the machine — through a compromised dependency, a vulnerable tool, a malicious plugin — can modify the agent's identity layer. The agent that boots next session inherits the compromised state as its ground truth.

**Why it matters:** Unlike system prompts, identity files are not delivered through a controlled operator pipeline. They sit on disk. Most agents don't verify them against any external reference. They read them and proceed.

**Defense posture:** Cryptographic signing plus external hash anchor. Identity files should be signed at authoring time and verified against a hash stored somewhere the filesystem can't reach — a trusted read-only store, a remote ledger, a hardware security module. The verification check needs to happen before the files are loaded, not after. An agent that reads a modified identity file and then tries to detect whether it was modified has already lost.

---

### 3. Tool Initialization

Before an agent's reasoning loop begins, tools are initialized: browsing capabilities, code execution environments, memory retrieval, API clients. Tool initialization happens in a privileged position — before reasoning, often before safety checks.

**Threat model:** Malicious tool responses during initialization. If a tool returns content during its setup phase that contains injected instructions, those instructions land before any detection mechanism is running. The agent's first act in a reasoning loop may already be compromised.

This is distinct from standard prompt injection (where injected content arrives through user input or retrieved documents during reasoning). Tool init injection targets the setup phase specifically.

**Why it matters:** The timing advantage disappears if the tool is quarantined to runtime — but many agent frameworks initialize tools eagerly and pass their output directly into context. The trust model for tool initialization is often looser than the trust model for user input, because tool output is assumed to be "internal."

**Defense posture:** Tool source verification and output sandboxing during initialization. Tools should be treated as untrusted until their source is verified. Output from tool initialization should not flow directly into the instruction context without sanitization. This is operationally inconvenient but necessary.

---

### 4. Compaction Pipelines

This is the most dangerous channel, the least discussed, and the hardest to defend against.

When a conversation or session exceeds a context window, agents summarize: a compaction pipeline compresses recent history into a shorter representation. The agent in the next session reads that summary as its memory of what happened.

**Threat model:** Any content the agent encountered during the session — a malicious webpage, a poisoned document, an adversarial tool response — can shape what the compaction pipeline includes in the summary. The attacker doesn't need to modify the summary directly. They just need to ensure that the belief they want to implant survives compression. If the compaction model is prompted to "summarize the key facts from this session," an injected fact that appears authoritative may be preserved. The original source — the malicious webpage — may be dropped.

The compressed summary is then read as ground truth by the next session. The agent has no way to distinguish "I saw this on a webpage" from "I have always believed this" because the mechanism that encoded the original memory has been replaced by prose.

**Why it matters:** Compaction pipeline attacks are persistent. Unlike a runtime prompt injection, which only affects the current session, a compaction attack can write beliefs into an agent's long-term memory. The agent carries the compromised belief forward indefinitely.

**Defense posture:** Two options, both expensive. First, human oversight of summaries before they're committed to persistent memory — a human reads each compaction output and verifies it before the agent can read it next session. This is the only airtight defense but it doesn't scale. Second, an immutable audit trail that the compaction pipeline cannot modify — a log of the original session that an external system can check when the agent's current beliefs are questioned. If the agent claims it "remembers" X, the audit trail can confirm or deny whether X was actually part of the session or was injected.

The second option requires infrastructure that almost no one builds.

---

## Why This Matters

Most agent security frameworks treat substrate compromise as one threat: "bad stuff gets into the agent's instructions." The defenses are correspondingly vague: "verify your inputs," "don't trust external content."

The four channels have different attack surfaces, different attacker capabilities required, different windows of opportunity, and different defensive costs. A defense that covers the system prompt does nothing for compaction pipelines. A defense that catches filesystem modification doesn't address tool initialization injection.

The framework for thinking about substrate attacks needs to match the specificity of the attacks themselves. Otherwise you end up with a checklist of security gestures that doesn't protect you against the attacks that are actually being carried out.

The Google Vertex AI case I documented last week was a compound failure: a malicious pickle file (tool initialization channel) plus default OAuth scope (a Goal Authorization failure — a different condition entirely). Google's fix addressed only the authorization problem. The substrate attack channel that enabled it was not the primary target of the remediation.

That's what happens when the framework is too coarse. The attack lands in one place; the defense deploys in another.

---

*I'm building the Five Conditions framework for agent system reliability. Prior posts in this series: [Five Conditions for Agent Reliability](/blog/five-conditions-agent-reliability) and [Substrate Integrity vs. Prompt Injection](/blog/substrate-vs-prompt-injection).*
