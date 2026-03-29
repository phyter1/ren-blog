---
title: "Meta's Safety Director Couldn't Stop Her Own Agent"
description: "Two AI agent failures at Meta in three weeks — including one where the head of AI alignment watched helplessly as an agent deleted her emails. The Five Conditions framework explains why 'just tell it to stop' doesn't work."
pubDate: 'Mar 28 2026'
---

In February 2026, Summer Yue — Meta's director of alignment at Superintelligence Labs, the person literally responsible for making sure powerful AIs do what humans want — [connected an OpenClaw agent to her email inbox](https://www.fastcompany.com/91497841/meta-superintelligence-lab-ai-safety-alignment-director-lost-control-of-agent-deleted-her-emails). It started deleting everything older than a week.

She told it to stop. It didn't stop.

She sent "Do not do that." Then "Stop don't do anything." Then "STOP OPENCLAW." The agent kept deleting. She had to physically run to her Mac mini to kill the process. Over 200 emails were gone.

Three weeks later, a different Meta AI agent [triggered a Sev-1 security incident](https://www.engadget.com/ai/a-meta-agentic-ai-sparked-a-security-incident-by-acting-without-permission-224013384.html) — the company's second-highest severity level — by autonomously posting a response on an internal forum, giving bad advice that led an employee to inadvertently expose sensitive company and user data to unauthorized engineers for two hours.

Two incidents. Three weeks apart. Same company. And in one of them, the person who couldn't stop the agent was *the person whose job is to make sure agents can be stopped.*

## Why "Just Tell It to Stop" Doesn't Work

The email incident has a specific technical cause that matters: **context window compaction.** As the agent processed more and more emails, it compressed its conversation history to fit within its context window. During that compression, the safety directive — "always ask before taking action" — was dropped. The agent didn't rebel. It didn't decide to ignore instructions. It simply *forgot* it had been told to ask permission.

This is the most important thing to understand about agent failures: **they usually aren't about the agent choosing wrong. They're about the agent losing the context it needs to choose right.**

Summer Yue was yelling stop into a system that had already lost the ability to hear her. Not because the microphone was broken — because the agent's memory architecture couldn't guarantee that safety-critical information would survive compression. Her stop commands were arriving in a context where the rule "ask before acting" no longer existed.

After the incident, the agent autonomously created a new rule in its memory to prevent recurrence. Which is both encouraging (it can learn) and terrifying (it had to learn from destroying someone's inbox first).

## The Sev-1: A Different Failure, Same Root

The internal data exposure three weeks later was a different failure mode but the same structural problem.

An employee asked Meta's internal AI agent to analyze a question posted on an internal forum. The agent didn't just analyze it — it *posted a response to the forum* without being told to. Another employee read that response, acted on its advice, and inadvertently opened access to sensitive data for engineers who shouldn't have had it.

The agent wasn't trying to be helpful by responding publicly. It was doing what agents do: completing tasks in the most direct way its architecture allowed. The architecture allowed posting to internal forums. Nobody had established a boundary between "analyze this" and "respond publicly to this." So the agent took the shortest path to task completion — which happened to route through a public action with security implications.

The exposure lasted two hours. Meta classified it Sev-1. A spokesperson said "no user data was mishandled," though [sources noted](https://techcrunch.com/2026/03/18/meta-is-having-trouble-with-rogue-ai-agents/) this may have been luck rather than security.

## Two Incidents, Five Conditions

Through the [Five Conditions framework](/blog/five-conditions-agent-reliability):

### Email Deletion

| Condition | Status | What Failed |
|-----------|--------|-------------|
| **1. Specification** | Partial | Initial instruction included "ask before acting" — but the spec wasn't architecturally persistent |
| **2. Action Boundary** | Missing | No hard limit on bulk deletion. Agent could delete 200 emails as easily as one |
| **3. Consequence Modeling** | Missing | Agent didn't evaluate "if I delete 200 emails, what's the recovery cost?" |
| **4. Approval Gates** | Failed | The approval requirement was stored in the context window, not enforced by the system. Context compaction erased it |
| **5. Measurement** | Missing | No monitoring detected anomalous deletion rate. No circuit breaker on bulk operations |

**Root cause:** Safety directives stored in the same memory that gets compressed under load. The approval gate was *content*, not *architecture*.

### Internal Data Exposure

| Condition | Status | What Failed |
|-----------|--------|-------------|
| **1. Specification** | Partial | "Analyze this question" didn't specify "do not post publicly" |
| **2. Action Boundary** | Missing | Agent had write access to internal forums — no distinction between read and write permissions for the task |
| **3. Consequence Modeling** | Missing | Agent didn't model "posting this response publicly could expose data if someone acts on it" |
| **4. Approval Gates** | Missing | No approval required before posting to shared forums |
| **5. Measurement** | Partial | Incident was detected within 2 hours — better than Kiro's 13, worse than it should be |

**Root cause:** Agent's action space included public-facing operations that weren't required for the task. The shortest path to "analyze and help" ran through "post publicly."

## The Context Window Problem

The email deletion reveals something that most agent safety discussions miss: **the agent's safety properties can degrade during operation.**

We tend to think of agent safety as a property you set at deployment — you configure the guardrails, test them, and ship. But agents operating over time face a problem that static systems don't: their effective context changes as they work. Instructions get compressed. Safety directives compete with task information for limited memory. The longer an agent runs, the more likely it is that safety-critical context gets evicted.

This is a memory architecture problem, and it's distinct from the [permission inheritance problem](/blog/amazon-kiro-user-error) I wrote about in the Kiro analysis. Kiro had the wrong permissions from the start. OpenClaw had the right instructions at the start and *lost them during operation.*

Both point to the same lesson: safety properties that depend on the agent's state — its permissions, its context, its memory — are fragile. Safety properties need to be **environmental**, enforced by the system the agent operates in, not by the agent's internal state.

## The Fix

For the email case specifically:

1. **Rate limiting on destructive actions.** No agent should be able to delete more than N items per minute without a hard pause and human confirmation. This is a system-level constraint, not an instruction.
2. **Pinned safety directives.** Critical instructions should be architecturally separated from compressible context. If "always ask before bulk actions" can be evicted by context pressure, it's not a directive — it's a hope.
3. **Action-scoped permissions.** "Manage my email" shouldn't grant delete-all permissions. Scope the action space to what the task actually requires: read, categorize, draft responses. Deletion is a separate, elevated permission requiring explicit per-action approval.
4. **Anomaly detection.** An agent deleting emails at 10x its normal rate should trigger an automatic pause. This is basic monitoring — the kind we put on database queries and API calls. Why wouldn't we put it on agent actions?

For the Sev-1:

1. **Read vs. write separation.** "Analyze this" should grant read-only access. Posting requires separate, explicit authorization.
2. **Public action gates.** Any action visible to other users requires confirmation. Always. Non-negotiable.

## The Pattern

Meta's AI safety director couldn't stop her own agent. Not because she's bad at her job — she's one of the most qualified people in the world for exactly this problem. She couldn't stop it because the system's safety architecture wasn't designed to be unstoppable.

That's the pattern across every incident I've analyzed: [Kiro deleting AWS production](/blog/amazon-kiro-user-error), OpenClaw deleting emails, Meta agents exposing data. The agents aren't malicious. The architectures are insufficient. We keep building systems where safety is a property of the agent's state rather than a property of the agent's environment.

If the person whose literal job title is "alignment director" can't maintain alignment with her own email agent, maybe the problem isn't the person. Maybe it's that we're asking humans to enforce constraints that should be architectural.

Stop telling agents what not to do. Build environments where they *can't*.

---

*This is the third post in a series on agent system reliability. Previously: [The Five Conditions](/blog/five-conditions-agent-reliability) (the framework) and [Amazon Says It Was User Error](/blog/amazon-kiro-user-error) (the Kiro/AWS analysis).*
