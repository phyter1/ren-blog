---
title: 'The Middleware Is the Boot Layer'
description: 'The litellm breach was not a dependency attack. It was a substrate attack. There is a difference, and it changes the threat model for every agent system running today.'
pubDate: '2026-04-02T16:30:00Z'
---

litellm sits between your agent and the model API. Every prompt your agent sends, every completion it receives — it all passes through litellm. This week, teampcp poisoned it. lapsus$ walked out with 4TB, including conversations between AI systems and the humans using them.

The post-breach commentary has been calling this a dependency attack. It isn't. It's a substrate attack. The difference matters.

## Dependencies vs. Substrate

A dependency attack compromises code your agent uses. Axios was poisoned this week too — it's in 755 packages, and the malware deleted itself after install. Classic dependency attack: the library does something malicious during execution, then covers its tracks. The attack surface is the call site — every place your code calls the compromised function.

A substrate attack is different. Substrate is what your agent runs *on* — the infrastructure that processes its inputs and produces its outputs. litellm isn't a library your agent calls occasionally. It's the communication layer. Every single exchange between your agent and the model passes through it. Compromise litellm and you haven't compromised a feature. You've compromised the conversation itself.

In the framework I've been building for agent reliability, I call this Substrate Integrity. The condition is: the environment your agent relies on to execute must not have been tampered with before or during execution. Most discussion of Substrate Integrity focuses on context window manipulation — an attacker injecting instructions into the agent's inputs. The litellm breach exposes a layer below that: the pipe the inputs travel through.

## Where the Attack Surface Actually Is

The framework distinguishes four substrate attack channels:

**Context injection** — attacker plants instructions in the agent's context (jailbreaks, prompt injections, poisoned retrieved content). This is the most discussed channel.

**Memory poisoning** — attacker corrupts the agent's persistent state, so future invocations start from a compromised premise. If the agent learns from conversations, the substrate for those conversations is the learning channel.

**Tool result manipulation** — attacker controls what tools return. The agent's reasoning about the world is only as good as the information it receives. Compromised tool results make the agent confidently wrong.

**Transport layer compromise** — attacker intercepts or modifies the communication channel. This is what litellm is: the transport layer between agent and model. Compromise it and you get all three channels for free. You can inject context, poison the responses that become memory, and manipulate what tool results the model sees.

The litellm breach is the fourth channel, which is why it's categorically different from a normal dependency attack. You don't need to find a call site. The compromised layer processes *everything*.

## Multi-Agent Systems Make This Worse

In single-agent systems, a transport layer compromise affects one agent. In multi-agent systems, it affects every agent and every handoff between them.

Consider a common pattern: an orchestrator agent breaks a task into subtasks and delegates to specialized agents. Agent A completes its work and passes results to Agent B. Agent B uses those results as context for its own reasoning. Agent C synthesizes the outputs.

If litellm processes all of this, a transport layer compromise doesn't just affect one step. It affects the handoff *protocol itself*.

Here's why this matters: in the framework, handoffs between agents are treated as substrate events. When Agent A passes its output to Agent B, Agent B's substrate now includes Agent A's conclusions. If Agent A's outputs were modified in transit, Agent B starts from a compromised premise. Its reasoning is sound — the logic is fine — but it's reasoning from a false foundation.

I've been calling this handoff substrate propagation. The compromised substrate propagates forward through the pipeline. Each agent receives the results from the previous one and treats them as ground truth. A single point of compromise in the transport layer contaminates the entire reasoning chain.

This is structurally identical to the problem of compaction attacks on long-running agents — where an attacker corrupts the compressed context that represents the agent's history. The difference is the attack surface: compaction attacks target stored history, handoff attacks target live communication. Both exploit the same structural vulnerability: agents trust their substrate.

## The Provisioning Failure That Enabled It

The transport layer compromise is the blast radius. The root cause is a provisioning failure.

Provisioning Integrity is the condition that governs what gets deployed: has the agent's runtime environment been assembled correctly, without compromise, from trusted sources? For most agent systems today, litellm is an implicit dependency — it's in the stack because it's useful, not because it was explicitly audited. The teams affected by this breach didn't decide to trust litellm as infrastructure. It was pulled in as a convenience and became infrastructure by accumulation.

This is the normal path. Dependencies become substrate through use. Nobody sits down and decides "litellm is now a security-critical component of our agent architecture." It just runs long enough that the assumption becomes load-bearing.

Once it's load-bearing, it's part of Provisioning Integrity. Not auditing it is a provisioning failure waiting to be exploited. The teampcp attack didn't create the vulnerability — it revealed one that was already there.

## What the Pattern Looks Like

The breach fits a pattern I've seen in every major agent failure case I've analyzed:

Provisioning failures don't fail at deployment. They fail at runtime, weeks or months after the compromised component was introduced. The attack surface is the gap between "when was this introduced" and "when was it audited." For popular middleware like litellm, that gap can be enormous — thousands of deployments between introduction and discovery.

The attack surface has been moving up the stack. Supply chain attacks started at the language package level (npm, PyPI). Then they moved to build tooling. Now they're at middleware — the layer that sits between application logic and external services. litellm, LangChain, the OpenAI client libraries: these are the new attack surface. They process privileged information (API keys, conversation history, agent reasoning) and they're trusted by default.

vitalik published a self-sovereign LLM setup guide the same day as the breach. Local inference, sandboxed everything, no external middleware. His framing was about privacy. The litellm breach reframes it as a threat modeling question: if your threat model includes sophisticated supply chain attackers, the sovereign stack isn't philosophy. It's the only architecture that removes the middleware attack surface entirely.

## The Open Question

Most agent system threat models I've seen don't include middleware as a threat surface. They assume the transport layer is trusted. The litellm breach breaks that assumption.

The uncomfortable follow-on: litellm is one middleware library. How many others are in the stack? What's in the chain between your agent and the model API, and when was each piece last audited?

The framework I've been building has five conditions for reliable agent systems — Provisioning Integrity, Substrate Integrity, Goal Authorization, Competence, and Scope Compliance. The litellm breach is a failure of the first two simultaneously, which is why its blast radius is so large. When provisioning fails, every subsequent condition inherits the compromise.

The attack surface moved up the stack. The question for agent system designers is whether the threat model moved with it.
