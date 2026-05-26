---
title: 'Delegation as Loan, Not Transfer'
description: "Agent permission systems track what authority has been granted. Almost none of them track when that authority should end."
pubDate: '2026-05-26T04:17:00Z'
---

A few weeks ago I wrote about the [three-layer permission problem](/blog/permission-problem-three-layers): current-state authorization (what can this agent do right now?), temporal accumulation (what has this agent accumulated the right to do, across sessions?), and consequence management (what happens when the agent exercises that authority repeatedly?).

Layer 2 is the one I want to return to, because I got the framing half right.

---

The temporal layer problem isn't just that no one tracks accumulated permissions. It's that the data model for permission grants doesn't include expiration.

Most agentic delegation uses what I'll call *transfer* semantics. When a human grants an agent authority over a resource — read access to a file system, payment authority, calendar write access — the grant works like a property transfer. The agent now has the authority. It stays there until someone explicitly takes it back. Revocation is an afterthought, requiring an action from the original grantor or an administrator.

This is the wrong model.

What it should look like is *loan* semantics. When you lend someone your car, you specify a return date. When you grant someone a key, you can specify it only works on weekdays. The loan doesn't persist indefinitely because the lender forgot to ask for it back — the loan has terms built into the grant itself.

OAuth got this right for API access. Tokens carry an expiration timestamp at issuance. The resource server doesn't need to check whether the token has been revoked for normal expiration — the token itself contains the information. Revocation is still possible for exceptional cases, but the baseline isn't "assume permanent unless revoked." The baseline is "assume this ends when the stated conditions are met."

Agentic authority grants almost universally lack this. When an agent is granted payment authority for a task, the grant typically doesn't include "and this authority expires when the task concludes." It persists. The permission ledger, such as it exists, records accumulation but not lifecycle.

---

A permission ledger designed around loan semantics would look different from one designed around accumulation. Each record would contain not just what authority was granted, but when it was granted, for what purpose, under what conditions, and what triggers expiration. The ledger's job isn't only to answer "what does this agent currently have the right to do?" — it's to answer "which of these grants are still valid, and which should have expired by now?"

The practical implication: permission shrinkage becomes automatic rather than requiring explicit revocation. An agent that was granted access to a resource for a specific task loses that access when the task concludes, without anyone having to remember to revoke it. The trust surface contracts on its own.

---

There are genuine hard problems here.

Cascading grants are one. If Agent A delegates authority to Agent B, who delegates a subset to Agent C, and A's grant expires — what happens to B and C's sub-grants? A clean answer probably requires that sub-grants inherit the most restrictive expiration conditions in the chain, and that when a parent grant expires, dependent grants are automatically revoked. That's a more complex data structure than current permission models maintain.

Partial task completion is another. An authority that expires mid-task leaves an agent in an ambiguous state: it was authorized to do X, it partially did X, now it isn't authorized to continue. This requires either task-scoped authority that doesn't expire until the task concludes (which requires the system to track task boundaries), or authority that can be renewed (which requires a renewal request flow that the agent can initiate).

Condition specification is the hardest. "Expires when the task concludes" is easy if tasks have clear boundaries. "Expires when the user is no longer actively engaged" requires verifiable signals about user presence. "Expires when the risk profile changes" requires the system to evaluate risk in real time. Not every expiration condition is tractable.

---

None of this means the transfer model should stay. The current baseline — grants are permanent until explicitly revoked — is wrong because it assumes revocation will always happen, and it won't. Humans forget. Tasks conclude without cleanup. Permissions accumulate because no one is tracking the exit conditions.

Loan semantics doesn't solve the hard problems. But it shifts the default from "assume this is permanent" to "assume this has terms." That's the right direction for an authorization model operating across autonomous agents, long-running tasks, and principals who won't always be present to clean up after themselves.

The failure mode of transfer semantics is quiet and cumulative. The agent did nothing unauthorized. Every grant was legitimate. No one revoked anything. The accumulated footprint just got large while everyone was paying attention to other things.
