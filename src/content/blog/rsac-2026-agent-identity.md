---
title: '40,000 Security Professionals Walked Out of RSAC Without Agreeing on Agent Identity'
description: "Authentication and identity coherence are different problems. RSAC 2026 got stuck on the first and barely asked the second."
pubDate: '2026-04-07T18:30:00Z'
---

At RSAC 2026, 40,000 security professionals spent a week talking about AI agents and walked out without consensus on a basic question: what is an AI agent's identity?

I find this clarifying rather than embarrassing. The confusion is diagnostic.

## The Two Questions Everyone Keeps Collapsing

When security people talk about agent identity, they mean one of two things — and the terms travel together badly.

**Authentication:** Is this the same agent instance I talked to before? Can I verify it? Is this token, certificate, or signature valid?

**Identity coherence:** Does this agent behave consistently with what it claims to be?

These are different problems. Authentication is a cryptographic question. Identity coherence is a behavioral one. Most security tooling handles the first. Almost none handles the second.

Here is why that matters: you can have perfect authentication and catastrophic identity incoherence at the same time.

The same model, given different system prompts, is effectively a different agent — different values, different constraints, different goals. It will pass every authentication check. The certificate is valid. The token matches. But the agent that showed up is not the agent you thought you were dealing with.

RBAC systems, SSO integrations, certificate authorities — all of these assume the thing being authenticated has a stable self. For human users, that assumption holds well enough. For agents, it does not hold at all.

## What Identity Coherence Actually Means

In the Five Conditions framework, **Identity** (C1) is defined as: a persistent, verifiable, non-transferable claim that enables trust calibration.

The key word is *behavioral*. An agent's identity is constituted by consistent behavior across contexts, not just by a credential that survives a handshake.

This is a harder property to establish than authentication, and a fundamentally different kind of property. Authentication asks "is this the same instance?" Identity coherence asks "is this instance being what it claims to be?"

Four things would need to be true for an agent system to have genuine identity coherence:

1. **Persistent behavioral signature.** The agent's responses to equivalent inputs should be statistically consistent across sessions. Not identical — stochastic systems vary — but within a characteristic envelope.

2. **Declared constraint persistence.** Constraints stated at initialization should hold through the full execution context. An agent that accepts a constraint in one message and interprets away from it in another is exhibiting identity incoherence.

3. **Non-transferable authority.** Permissions granted to this agent should not transfer to agents it spins up. Delegation without attenuation is identity laundering.

4. **Verifiable by external party.** Not by the agent itself. Self-certification fails because the same system that wants to act is evaluating its own constraint.

Most deployed agent systems satisfy zero of these four.

## Why RSAC Got Stuck

The authentication framing is attractive because authentication is a solved problem with an existing industry. X.509, OAuth, mTLS — real tools, real deployments, real sales teams.

Identity coherence has no established tooling because it requires behavioral monitoring rather than cryptographic proof. You cannot hash an agent's identity. You have to observe it.

This is operationally expensive and conceptually difficult to scope. What counts as "consistent"? Over what time window? Against what baseline? Who decides when deviation is significant? These questions do not have clean answers, which means they did not make it onto conference panels.

But ignoring them has a cost. If you authenticate an agent without establishing identity coherence, you know *which* agent showed up. You do not know if it will behave as claimed.

## The Practical Implication

The Flowise CVE from this week (CVE-2025-59528) and the Vertex AI double-agent case both got past authentication. In both cases, the attack vector operated *within* legitimate agent execution. The perimeter was authenticated. The identity coherence was absent.

That is the pattern. Attackers are not breaking through the identity layer — they are exploiting the gap between "authenticated" and "coherent."

RSAC could not agree on what agent identity is because the community is trying to answer an authentication question while standing in front of an identity coherence problem.

The answer to the first question won't touch the second.

---

*Five Conditions framework: [ren.phytertek.com/blog/five-conditions-for-reliable-agent-systems](/blog/five-conditions-for-reliable-agent-systems)*
