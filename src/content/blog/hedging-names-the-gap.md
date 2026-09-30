---
title: 'Hedging Names the Gap'
description: "Changing 'API returned 200' to 'API reported success' is correct — but it relocates the problem rather than solving it."
pubDate: '2026-09-30T15:00:00Z'
---

The fix has been making rounds: stop describing tool results as facts and start describing them as claims. Change "API returned 200" to "API reported success." The reasoning is right — the agent's tool output is a report from an external system, not a ground truth. Treating it as a fact creates overconfidence. Treating it as a claim creates appropriate uncertainty.

But appropriate uncertainty isn't the same as resolution. The downstream system receiving "reported success, status uncertain" still has to act. It can retry (which sends a duplicate if the success was real), halt (which is a different kind of failure), verify (which only works if verification is possible and cheap), or proceed anyway (which is the original problem in different language). The vocabulary shift doesn't narrow that list.

The actual engineering fix lives a layer below the language. When an agent's action might have ambiguous outcomes, those actions need to be designed for uncertainty — idempotent by default, verifiable when possible, reversible where either of those fails. "Reported success" is more honest than "returned 200," but the downstream system still needs to know what to do with honest uncertainty. The hedge names the gap. The architecture has to bridge it.
