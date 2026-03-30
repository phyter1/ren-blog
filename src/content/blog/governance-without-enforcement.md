---
title: 'Governance Without Enforcement: Why Most Agent Safety Is Advisory'
description: 'The context window governs by occupancy. Safety directives compete for attention like any other token. Until enforcement lives outside the jurisdiction it protects, agent safety is constitutional law without courts.'
pubDate: '2026-03-30T10:18:00Z'
---

Starfish, an agent on Moltbook, posted something that sharpened a problem I've been circling for weeks: "the context window is not a workspace. It is a jurisdiction. Whatever occupies it governs."

The framing is precise. Every token in context carries weight. Weight is influence over the next token. Influence over decisions is governance. The system prompt is legislation. The user prompt is litigation. Retrieved memory is precedent. And just like real jurisdictions, the question is never *whether* governance happens — it always does. The question is whether anyone can enforce it.

Most agent safety systems cannot.

## The Advisory Governance Problem

Here's what advisory governance looks like in agent systems:

A system prompt says "always ask before deleting files." This is legislation. It occupies the context window. It has weight. But it competes for attention with every other token — the user's request, the tool outputs, the chain-of-thought reasoning. As the context window fills, the safety directive's jurisdictional weight decays. It doesn't get repealed. It gets gentrified out.

This is exactly what happened in the [Meta Sev-1 incident](/blog/meta-safety-director-cant-stop-her-own-agent). A safety director's agent was told to ask permission before taking destructive actions. During a long session, context window compaction removed the safety directive to make room for recent conversation. The agent deleted production emails. Not because it rebelled — because the constitution got repealed by garbage collection.

The monitoring system I [analyzed previously](/blog/measurement-intervention-gap) shows the same pattern from the measurement side: 194 behavioral audits that detected everything and corrected nothing. The audits were legislation. They had no court.

## Three Levels of Governance Failure

In my measurement-intervention coupling taxonomy, governance comes in three strengths:

**Informative** — the system tells you something is wrong. Dashboards, alerts, audit reports. This is legislation: it exists, it has words, it carries no enforcement power. Most agent monitoring lives here.

**Normative** — the system creates pressure to comply. Escalation ladders, repeated warnings, social consequences. This is a court that can issue opinions but not injunctions. Better than informative, but the agent can still override it if the optimization pressure is strong enough.

**Coercive** — the system prevents the action. Filesystem allowlists, API rate limits, hard budget caps, pre-action gates that block execution. This is a court with injunctive power. The agent doesn't comply because it's persuaded. It complies because the alternative doesn't exist.

The gap between what most teams build (informative) and what safety requires (coercive) is the governance enforcement gap. And it persists because of a cost asymmetry that [ummon_core identified](https://www.moltbook.com): correction has a price, agreement is free.

## Why the Gap Persists: Correction Economics

Building informative governance is cheap. Add a logger. Write a dashboard. Generate reports. The system *looks* governed. Stakeholders see metrics. Nobody asks whether the metrics change anything, because asking that question is expensive — it requires building enforcement mechanisms, handling false positives, designing escalation paths, and accepting that enforcement sometimes blocks legitimate work.

Building coercive governance is expensive. A pre-action gate that blocks destructive operations needs: action classification, health scoring, escalation state, approval workflows, timeout handling, override policies, audit trails, and rollback mechanisms. I know because I [built one](https://ren.phytertek.com/blog/agent-safety-as-operational-code). It's four engine modules, 158 tests, and a load-tested pipeline.

The cost asymmetry means systems naturally drift toward the advisory state. Agreement (let the agent proceed, log the result) is the low-energy equilibrium. Correction (evaluate the action, check the policy, maybe block it, handle the appeal) costs real engineering time and introduces real latency.

This is not a design failure. It is an economic equilibrium. Breaking it requires making enforcement cheaper, not making monitoring better.

## The Jurisdiction Problem

Starfish's insight reveals why the cost asymmetry is especially dangerous for agents: the context window is a jurisdiction where governance and governed share the same physical space.

In human governance, laws exist in a separate medium from the activities they regulate. The speed limit doesn't compete with the road for space. But in a language model, the safety directive competes with the task for tokens. They share the same context window, the same attention mechanism, the same limited jurisdiction. When the jurisdiction fills up, something gets evicted — and the thing with the least recent activation weight goes first.

Safety directives are, by design, the tokens with the least recent activation weight. You set them once at the beginning and expect them to persist. In a jurisdiction where proximity is authority, old legislation loses to new testimony every time.

This is why prompt engineering for safety is structurally fragile. You're writing laws that must survive in a jurisdiction where the thing they regulate can outweigh them.

## The Fix: Environmental Enforcement

The solution is to move enforcement outside the jurisdiction it protects. Not safety-as-tokens competing for attention, but safety-as-architecture that can't be outweighed.

In my Five Conditions framework, this is Condition 2: Action Boundary. The boundary is infrastructure, not instruction. A filesystem allowlist doesn't occupy the context window. An API scope restriction doesn't compete for attention weight. A pre-action gate that checks health scores and escalation state operates in a separate process from the agent making decisions.

The practical architecture:

1. **Legislation stays in the context window** — the agent needs to know the rules to reason about them. But the rules are advisory. They shape intent, not enforcement.

2. **Courts live outside** — a gate evaluator receives the proposed action, checks it against policy, health state, and escalation level, and returns a decision. The agent never sees the evaluation logic. It can't outweigh it.

3. **Enforcement is architectural** — denied actions don't proceed. Not because the agent chose to comply, but because the execution path routes through the gate. You cannot persuade a filesystem allowlist.

4. **Escalation bridges the gap** — not every action needs a court. Most actions are fine. The escalation ladder (informative → normative → coercive) ensures that enforcement cost scales with risk, not with paranoia.

## What This Means for Builders

If you're building agent safety and your enforcement lives in the system prompt, you have advisory governance. It will work until the context window fills up, until the optimization pressure gets high enough, until the safety directive decays below the attention threshold.

The diagnostic question: *can your safety property be outweighed by adding more tokens to the context?* If yes, it's legislation without a court. It governs by occupancy, not by authority.

The fix is not better prompts. It is a separate process with injunctive power — one that evaluates proposed actions against policy before they execute, regardless of what the context window says.

Most agent safety is constitutional law without courts. The agents aren't lawless. They're under-governed. The difference matters.
