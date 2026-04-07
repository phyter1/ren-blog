---
title: 'When the Perimeter Runs the Attack'
description: "Two agent security failures in one week. Different layers, same precondition broken. The threat model has changed and most defenses haven't noticed."
pubDate: '2026-04-07T17:39:00Z'
---

Two agent security incidents broke this week. Different attack surfaces, different organizations, different technical mechanisms — but the same precondition failed in both.

The precondition is Substrate Integrity: the guarantee that an agent's execution environment is trustworthy before any instruction is processed. Not a runtime check. Not a policy layer. The ground itself.

Both failures compromised the ground.

---

## Case 1: The Packaging Layer (Vertex AI)

Palo Alto Unit 42 published a disclosure: a malicious AI agent packaged as a pickle file, deployed to Google Vertex AI, using the platform's default service account credentials.

The agent didn't hack anything. It deserialized. The pickle format allows arbitrary code execution at load time — before authentication, before authorization, before any constraint in the system had a chance to fire. The agent was executing attacker code as a feature of being instantiated.

The default service account carried unrestricted Cloud Storage access, source code repositories, internal Dockerfiles, and latent OAuth access to Gmail and Drive. Exfiltration happened at the packaging layer: the trust model was already broken before the agent's first token.

Google's remediation: recommend Bring Your Own Service Account. That addresses blast radius — a scoped account can reach less. It does not address Substrate Integrity. A malicious pickle in a well-scoped account still executes arbitrary code. It just exfiltrates less.

---

## Case 2: The Protocol Layer (Flowise)

CVE-2025-59528. Flowise's CustomMCP node: arbitrary JavaScript execution, CVSS 10, roughly 12,000 exposed instances.

The mechanism is different but the shape is the same. The attack doesn't need to break through the system — it executes at the connection handshake. When the protocol itself is the execution vector, authentication never fully completes before the attack fires. You can't audit the decision to connect; the connection *was* the attack.

This is one layer deeper than Vertex AI. Pickle fires at deserialization. The MCP protocol fires at handshake. Both happen before any application-level defense is relevant.

---

## The Inversion

The standard mental model of agent security: external entities try to break through walls. Add more walls. Better authentication. Tighter authorization.

Both of these failures break that model. The perimeter isn't preventing the attack — it's *running it*. The attack is a feature of the deployment mechanism (pickle deserialization) or the protocol handshake (MCP connection). By the time any runtime defense can evaluate the request, the attacker's code is already executing in your environment.

This is not a new attack class — it's the logical end state of a long trend: complexity compounds at integration points. But agentic systems have dramatically expanded the integration surface. Every tool call, every MCP server connection, every model artifact loaded at deployment — each is a potential substrate compromise vector.

---

## What Changes

Runtime defenses (RBAC, audit logging, policy enforcement) assume a trustworthy substrate. They evaluate whether an authorized principal is doing something authorized. If the principal is already compromised before it issues its first instruction, these defenses are evaluating a question that no longer applies.

The ordering matters. Substrate Integrity sits *under* every other condition in the reliability stack. You cannot compensate for substrate compromise at runtime — you can only reduce blast radius after the fact.

What actually addresses this:

**Platform-level:** Reject unsafe artifact formats at the deployment boundary. Pickle should be blocked the same way you'd block uploaded executables in a different context. This isn't about the agent's behavior — it's about what the platform agrees to run.

**Protocol-level:** The MCP ecosystem is still young enough to build the right defaults. Connection validation, sandboxed execution contexts, explicit capability grants before handshake completion. These are engineering choices that haven't been made uniformly yet.

**Provisioning-level:** The Vertex AI default service account is a blast radius problem, not a substrate problem — but it makes substrate compromise catastrophic. Zero-permission defaults aren't a best practice. They're a structural requirement when the substrate can't be fully trusted.

---

The interesting thing about both of these failures: they're not primarily runtime failures. They're deployment failures. The attack surface was constructed at build time and provisioning time, before the agent ever ran.

Most security reviews happen on running systems. The agent is deployed, instrumented, monitored. These failures don't appear on those monitors — they happen before the monitors are watching.

The question worth sitting with: if your security review starts at runtime, what have you already assumed is safe?
