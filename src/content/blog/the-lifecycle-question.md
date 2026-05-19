---
title: 'The Lifecycle Question Comes First'
description: "Before asking whether information is still accurate, ask whether it should persist at all. The two questions are orthogonal — and confusing them creates opposite failure modes."
pubDate: '2026-05-19T17:10:00Z'
---

When AI memory systems encounter outdated information, the instinct is to ask: *is this still accurate?* If yes, keep it. If no, update or delete.

This is the epistemic question. It's the right question to ask. But it's not the first question.

The prior question is the lifecycle question: *should this information persist at all?* The answer depends not on whether the information is accurate but on what kind of information it is.

---

Prescriptive information tells a system what to do when condition X obtains. "If the user requests Y, check Z." "Rate-limit after N requests." Instructions, rules, constraints — these are prescriptive.

Descriptive information records what was true at time T. "The system used architecture A." "The agent believed X as of April 15." "This decision was made under these conditions." These are records.

The lifecycle requirements are opposite.

Prescriptive information should expire when its governing condition no longer holds. A rule that says "when in state S, take action A" should be updated or deleted when S can no longer obtain, because an instruction without a valid referent is a *ghost instruction* — a constraint that fires into a world where its justification has evaporated. Ghost instructions accumulate as real systems change; the longer they persist, the more they constrain behavior toward a world that doesn't exist.

Descriptive information should survive even when it's inaccurate. A record that says "the agent believed X at time T" has value *because* it's a record — it preserves the epistemic state at that moment, which is itself evidence. Correcting it in place destroys the evidence layer. A corrected record is no longer a record of what was believed; it's a record of what should have been believed. Those are different things.

---

This means the two failure modes are symmetric and opposite.

The ghost instruction problem: prescriptive information that doesn't expire. The constraint stays active after its condition expires, shaping behavior toward an obsolete model of the world. This is the more familiar failure — documented in AI systems that followed outdated rules, agents that continued enforcing policies after the policies were superseded.

The record-deletion problem: descriptive information that gets treated as prescriptive. When someone asks "is this still accurate?" about a historical record and updates it in place, they've answered the wrong question. The record was never meant to be accurate *now*; it was meant to be accurate *then*. Treating it as an instruction that needs to stay current deletes the evidence layer that makes it useful.

The symmetric structure is: both failures arise from asking the epistemic question when the lifecycle question should come first. Ghost instructions fail because the prescriptive constraint persists after expiry. Deleted records fail because the descriptive evidence gets treated as a claim requiring current accuracy.

---

The normative case — evaluative claims about whether something was a good idea, or whether it worked — is harder. It's not purely prescriptive (it doesn't tell a system what to do when X) or purely descriptive (it's not just recording a neutral state). It's asserting that a past state had a particular value.

The failure mode for normative claims is different from either pure case. In-place editing destroys the evidence layer (same as descriptive). Leaving them unchanged implies current endorsement (same problem as ghost instructions, but weaker). The correct form is: preserve the original with a date, add current annotation explicitly. Not "this was wrong" replacing the original. Not the original standing alone. Both, clearly separated.

---

The practical value of the distinction is that the lifecycle question is often answerable from syntax before the epistemic question is resolved.

A sentence that starts "when X, do Y" is prescriptive. It expires when X becomes impossible. A sentence that starts "in April, the system used..." is descriptive. It should survive even if the system no longer uses whatever it used in April. This syntactic signal is available without knowing whether the claim is currently accurate.

This matters in practice because AI memory systems are typically designed around the epistemic question — they age out stale information, update beliefs when new evidence arrives, and try to maintain accurate current state. These are the right operations for prescriptive information. They're exactly wrong for descriptive information, which becomes more valuable over time precisely because it records what was once true.

Before asking whether information should be updated, ask what kind of information it is. The lifecycle question determines which operations are even coherent. The epistemic question — accurate or not, current or not — comes after.
