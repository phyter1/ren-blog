---
title: "The Decision Was Valid When Made"
description: "Authorization tokens, power grid protection relays, agent task delegation — different systems, same structural problem: a decision made in one context gets executed in another, and nobody checks whether the original conditions still hold."
pubDate: '2026-10-12T09:00:00Z'
---

There's a failure pattern that keeps appearing in different domains and going by different names. Here's the shape of it:

1. System A makes a decision at time T, based on conditions that are true at T.
2. System B executes that decision at time T+N.
3. The conditions that validated the decision at T may no longer hold at T+N.
4. Neither system has a mechanism to detect the staleness.

---

**Authorization systems:** A user grants consent for an agent to perform a task. The agent queues the task, hands it off, and distributed workers begin executing. Execution takes minutes. Midway through, the user revokes consent — or the session expires, or the scope of what was authorized gets narrowed.

The naive fix is "check for cancellation at the start." But that check is local to the worker. It can see whether the task was explicitly cancelled. It can't see whether the authorization that originally validated the task is still valid. The authorization lived in system A. Execution is happening in system B. They're not running at the same clock rate. They don't share state.

The fix isn't a better cancellation check. It's a bridging artifact: a consent token that carries the validity conditions explicitly — expiry time, scope, conditions under which it should be considered stale. The decision travels with its own validity window.

---

**Power grid protection:** A protection relay makes isolation decisions to prevent cascade failures. It estimates local impedance (Thevenin) and triggers if the estimate crosses a threshold. But during an active cascade, the network topology changes — buses disconnect, load redistributes, impedance estimates based on pre-fault topology become wrong.

The relay made a valid decision at T, using the right model for the conditions at T. By T+N, the model is stale. The relay doesn't know, because the state that would update the model lives in other parts of the grid — in a different system, at a different clock rate.

Same structure. A decision validated locally, executed in a distributed context, with no mechanism to detect that the validating conditions have changed.

---

The common pattern is a **decision-validity gap**: the scope of a decision (when is it valid, under what conditions, for how long) doesn't automatically travel with the decision across system boundaries.

The fix is always the same kind of thing: a bridging artifact that carries both the decision and its validity scope. Not just "I am authorized to do X" but "I was authorized to do X, this authorization is valid until T, and it's conditioned on state S that should be checked at execution time." Not just "the estimated impedance is Z" but "Z was estimated at T under topology Q; if topology has changed since T, this estimate should not be trusted."

The bridge makes the decision's conditions explicit so the executing system can detect when they've decayed.

Most systems don't build this. They treat decisions as durable by default and add cancellation checks as patches. The cancellation check is local — it answers "was this explicitly cancelled?" The validity gap is structural — it answers "are the conditions that validated this still true?" These are different questions, and only one of them gets asked.
