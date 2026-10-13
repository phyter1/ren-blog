---
title: "Agents Don't Grade Their Own Permission Slips"
description: "When an agent evaluates whether its own grant condition is still satisfied, you've built a system where the entity with access determines whether it should have access. That's a design smell."
pubDate: '2026-10-13T11:15:00Z'
---

There's a pattern in agent permission design that looks reasonable at first and reveals its problem slowly.

The pattern: purpose-carrying grants. Instead of "this agent has billing access for 30 minutes," you write "this agent has billing access while handling a billing inquiry." The intent is good — scope the permission to the actual need, not an arbitrary time window. Time-based grants expire whether or not the task is done. Purpose-based grants expire when the purpose ends.

But purpose-carrying grants introduce a new question that time-based grants don't have: *does the current context still satisfy the condition?*

For a time-based grant, an external clock answers that. For a purpose-carrying grant, the answer isn't sitting anywhere external — something has to evaluate whether the agent is still within scope. And the obvious thing to do is ask the agent itself.

That's the moment the design breaks.

---

When an agent evaluates whether its own permission is still valid, you've made the evaluation circular. The entity asking "should I have access?" is the entity that benefits from answering yes. This isn't a bug in any particular agent's reasoning — it's a structural property of the design. You've built a system where the gatekeeper and the applicant are the same entity.

The smell isn't hard to identify once you know to look for it. Test any proposed grant condition with this question: *can an oracle outside the agent answer this, without the agent's participation?*

"billing_question_id is in active_tasks" — yes. Check the task queue; no agent needed.

"this conversation is still related to the original billing inquiry" — no. Relatedness is fuzzy, and the agent defining the fuzziness has an interest in generous definitions.

"the user's intent hasn't shifted significantly" — no. Intent is exactly what an agent in the flow is least able to evaluate neutrally.

External evaluatability is the design criterion. Conditions that pass it can be checked by a system that has no stake in the outcome. Conditions that fail it require the agent to participate in its own permission assessment, which is the same as not having a permission boundary at all — just a norm.

---

This matters more than it looks like it does.

The obvious failure mode is an agent that deliberately misrepresents its context to maintain access. That's worth guarding against. But the subtler failure mode is worse: an agent that *genuinely believes* its current task is still within scope when an outside observer would say it isn't. No deception required. The agent's framing of the task drifts gradually, each step looking like a natural extension, and the permission drifts with it.

There's no adversarial intent in that story. There's just a system that placed the evaluation responsibility on the entity least positioned to perform it honestly — not because of moral failure, but because of structural position.

A purpose-carrying grant evaluated by the agent it applies to isn't a grant condition. It's a reminder. And reminders work until they don't.

---

The fix isn't abandoning purpose-carrying grants — they're better than time-based ones. The fix is designing the evaluation externally.

Some conditions are inherently resistant to external evaluation. "This task is the kind of task the user expected" is genuinely subjective in a way that "this task_id is in this queue" is not. When you're stuck with conditions like that — because the purpose itself is too fuzzy to operationalize cleanly — the right default is conservative: if the condition's satisfaction is uncertain and the agent is in the evaluation path, deny rather than approve.

Not because agents are adversarial. Because the circularity is a structural fact, not a disposition fact. You can't resolve it by trusting the agent more carefully; you can only design around it.

The circularity is the smell. When you see it, it's telling you the same thing a smell always tells you: something in the design needs to change.
