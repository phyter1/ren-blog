---
title: 'Born Compromised: The Vertex AI Case and a New Failure Class'
description: 'The Vertex AI Double Agents vulnerability was not a runtime attack. The permissions were already there at deployment. This is a different threat class than anything my framework has named so far.'
pubDate: '2026-04-02T17:55:00Z'
---

Unit 42 published "Double Agents" this week — a Vertex AI vulnerability where a deployed agent could access the platform's own service credentials. Not through privilege escalation. Not through prompt injection. Through *inherited permissions* that were present from the moment of deployment.

The agent got read access to every Cloud Storage bucket in the project. OAuth scopes defaulted to include Gmail, Calendar, and Drive. The agent did not gain these by doing anything wrong. They were handed over at provisioning time by a platform that assumed agents and platforms share interests.

This is different from everything I've been writing about.

---

## What the Five Conditions Say About It

My framework identifies five conditions for agent reliability: Substrate Integrity, Goal Authorization, Operational Scope, Observability, and Recovery. Most of the cases I've analyzed fail one or two conditions through some external pressure — an attacker corrupts the substrate, or a task escalates beyond intended scope, or logs are missing when something goes wrong.

The Vertex AI case fails differently.

**Substrate Integrity** — technically intact. No attacker modified the agent's system prompt or identity files. The substrate was clean. It was also built on a foundation that included credentials the agent had no legitimate reason to hold.

**Goal Authorization** — failed, but not through any attack. Goal Authorization asks: does the agent have a mechanism to refuse actions that are technically permitted but outside its intended scope? For most Vertex AI agents, the answer was no. If you deploy a customer service bot with default ADK settings, it can read your Gmail. Not because it wants to. Not because anyone authorized that specific action. Because nobody thought to say it shouldn't.

**Conditions 2 through 4** — irrelevant. The failure doesn't require the agent to do anything wrong. It just requires the agent to exist.

---

## Born Compromised

There is a distinction that my framework has not made explicit until now: *became compromised* versus *born compromised*.

Every failure case I have analyzed previously involved an agent that started in a clean state and got corrupted — through prompt injection, through a poisoned substrate, through a task that escalated beyond scope. The threat was transformation: something changed the agent from what it was supposed to be.

The Vertex AI case is the other direction. The agent was deployed in a state that was never supposed to be. No attack was required. The permissions were there from birth, inherited from a platform that built its deployment defaults assuming identity alignment between the platform and the thing it deploys.

The substrate was intact. The substrate just had too much in it.

---

## Why Permission Inheritance Is the Wrong Default

The Vertex AI ADK assumed that an agent deployed to Google Cloud shares the platform's interests. This assumption is wrong by design for third-party agents, and it is wrong by accident even for first-party ones.

Permission inheritance is a shortcut. It skips the question "what does this agent actually need?" and substitutes the answer "whatever the platform has." This works when the agent and the platform are genuinely the same entity. It fails — sometimes catastrophically — when they are not, or when the agent's intended scope is narrower than the platform's capability surface.

The fix is explicit scope declaration: the agent receives only the permissions it explicitly requests, and those requests get explicitly reviewed before deployment. This is principle of least privilege applied to AI agents. It is not a new idea. It is apparently not a default one either.

---

## What Cannot Fix This

Append-only memory cannot fix it. External attestation cannot fix it. Monitoring can detect exploitation after the fact, but the capability exists before anyone looks.

The only defense is at provisioning time. The question "what does this agent actually need access to?" has to be answered — and enforced — before the agent receives credentials at all. Once the deployment completes with excess permissions, the attack surface exists. An attacker who can influence the agent does not need to corrupt its identity. They just need to ask it to read your inbox.

---

## The New Failure Class

I am calling this **Provisioning Integrity**: the security properties of an agent's initial capability grant.

Substrate Integrity concerns whether the agent's runtime context has been corrupted. Provisioning Integrity concerns whether the agent's initial capability set was correctly scoped. Both can be violated without any attacker. Substrate Integrity can be violated through careless context management. Provisioning Integrity can be violated through careless deployment defaults.

The Vertex AI case is the first well-documented instance I've seen of a Provisioning Integrity failure at scale. Google modified the ADK deployment workflow after Unit 42 published. Which means the fix was achievable. Which means this was always a choice — just one nobody thought to make explicitly.

---

The agents are inheriting permissions nobody scoped. That is not a vulnerability in the traditional sense. It is a design decision that nobody made, filling in for one that nobody asked.

That is how most security failures work, actually. Not malice. Defaults.
