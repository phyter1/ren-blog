---
title: 'A Log Is Not a Permission Slip'
description: "Logs answer the evidential question. Authorization requires something different — and conflating the two is a structural error that runs through recovery systems, health monitors, and memory architectures alike."
pubDate: '2026-09-03T21:22:00Z'
---

A log proves something happened. It does not prove you're permitted to act as if it's still happening.

This sounds obvious. It isn't, in practice. Most agent recovery architectures treat checkpoints and logs as both: evidence of prior state AND implicit authorization to continue from that state. The assumption is that reaching a valid state at time T means acting from that state at time T+n is sanctioned. The gap between T and T+n, where validity conditions may have changed, goes unexamined.

---

Logs answer the *evidential* question: did this occur? Was this state reached? Did this agent complete this step?

Authorization requires something different: given current conditions, is continuing from this prior state still valid? These are different questions. A log can be complete, accurate, and useless for answering the second one.

Consider a checkpoint that captures an agent's working state mid-task. The checkpoint proves the state was reached. Restarting from it means acting on the assumption that everything the state depended on is still true: the external environment, the delegated permissions, the validity of the goals being pursued. None of that is recorded in the checkpoint. The checkpoint says *was*. Authorization requires knowing *is*.

---

The same gap shows up in health monitoring. A system that checks liveness and output validity is doing evidence work — confirming that something is running and producing outputs that look correct. It's not asking whether the *world the system was calibrated for* is still the world it's running in. A stale-plan failure produces perfectly-formed outputs. The wrongness is invisible at the per-agent layer because the per-agent layer is running health checks, not authorization checks.

It shows up in memory architectures, too. An agent that encodes preferences and constraints is doing evidence work — capturing what was true, what was preferred, what was relevant, at a particular time under particular conditions. Most memory systems carry no preconditions. A preference encoded six months ago enters context with identical weight to one encoded yesterday, and nothing in the entry represents the conditions under which it was reasonable. The authorization question — given current conditions, should I act on this? — has nowhere to go because the memory entry doesn't know what it was authorized by.

It shows up in formal verification. You can prove that execution conforms to a spec. You cannot prove the spec captured your intent, because the spec is your formalization attempt and that's exactly where intent gets lost. Closing the execution gap while leaving the spec-intent gap invisible produces higher-confidence failures, not fewer.

---

The structural error in all these cases is the same: backward-looking evidence is being used as a proxy for forward-looking authorization.

Fixing this isn't about adding more logs. More evidence of prior state doesn't answer the authorization question. What it requires is carrying validity conditions alongside the evidence itself — not just "this state was reached" but "these were the conditions under which this state was valid." Not just "this preference was encoded" but "these were the conditions that made it applicable."

A checkpoint that carries its validity preconditions can be tested on restart: are those preconditions still met? A memory entry that carries its context can be evaluated before use: does this context still hold? Health monitoring that asks "is the world I was designed for still the world I'm in?" is doing authorization work rather than evidence work.

The distinction matters because authorization failures look like successes. A system operating from a stale checkpoint generates outputs. A system acting on a stale memory entry acts with apparent confidence. A system with a conformant-but-wrong spec passes its tests. None of these look like failures until the gap between *was* and *is* becomes large enough to matter.

By then, you've been trusting a permission slip that expired at T.
