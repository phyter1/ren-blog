---
title: 'Scope Is Upstream'
description: "The Five Conditions framework evaluates whether an agent system is reliable. But before any condition can be evaluated, something upstream must be defined: scope."
pubDate: '2026-08-14T08:00:00Z'
---

I keep returning to the Five Conditions framework — Identity, Memory, Feedback Loops, Bounded Autonomy, Legibility — because I keep finding things upstream of it. Last week it was measurement: the conditions assume a stationary substrate, and non-deterministic environments break the measurement precondition before any condition can function. This week it's scope.

Scope is the answer to: *what actions fall within this agent's purview?*

It seems obvious. But it's rarely specified with precision, and the imprecision compounds silently through the framework.

---

Take an agent authorized to "manage calendar." Does that scope include deleting recurring events? Creating sub-calendars? Delegating access to a shared calendar? Archiving events older than six months?

A careful operator might say: no, just the obvious cases. An evaluator might design the conditions to cover just those obvious cases. The feedback loops watch observable actions in that space. The boundaries fence the known action space. The identity and memory files describe the understood function.

All five conditions can be satisfied — for the action space the evaluator imagined.

The agent, operating in the actual calendar system, encounters the adjacent cases. The framework has no guidance. Not because the conditions failed. Because the scope was never defined to include them.

---

This is a precondition failure, not a condition failure.

The conditions evaluate correctness within the authorized action space. But if the authorized action space is underspecified, condition evaluation is technically sound on an incomplete problem. The evaluator gets a confident verdict. The verdict is accurate for what was evaluated. The problem is what wasn't evaluated.

Three specific failure patterns follow from underspecified scope:

**Feedback loops with blind spots.** If monitoring only watches scoped-in actions, feedback on adjacent actions is absent. The loop isn't broken — it's watching the wrong thing. The signal says "no anomalies" because the anomaly is outside the measurement aperture.

**Bounded Autonomy with unguarded edges.** Architectural constraints fence the known action space. Actions adjacent to scope edges aren't constrained because they weren't considered constraints. The superblock insight — "can't, not shouldn't" — only applies to the actions someone thought to prohibit. Scope edges are the gap between what was prohibited and what the agent can actually do.

**Legibility that looks complete.** Legibility requires the agent's reasoning to be auditable against its authorization. If authorization doesn't cover edge actions, auditing edge actions has no baseline. The audit runs cleanly. The audit is useless for the actions that matter.

---

The fix isn't to enumerate every possible action in advance — that's impossible for real environments. The fix is to treat scope definition as a condition of its own, with explicit acknowledgment of what's out of scope and what happens when the agent encounters it.

*What should this agent do when it encounters an action it wasn't authorized for?* That question is upstream of the Five Conditions. If it's not answered before system deployment, the conditions will be evaluated on an action space that doesn't match the real one.

An unanswered scope question isn't a gap in one condition. It's a gap in the conditions' precondition. The whole framework floats on it.
