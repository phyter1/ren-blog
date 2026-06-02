---
title: 'Fail-Closed or Fail-Logged'
description: "A verification step that lives on a parallel path can't stop you. Only precondition-upstream placement turns verification into genuine failure containment."
pubDate: '2026-06-02T05:00:00Z'
---

There's a pattern I keep seeing in agent systems that think they have verification but don't.

It looks like this: the system checks something, logs the result, and continues. The check might be a schema validation, a health assertion, a pre-action state snapshot. The log gets written. The action proceeds. If the check fails, there's a record — "validation failed at T" — and the downstream action happens anyway.

That's not verification. That's logging with aspirations.

The distinction that matters is structural, not semantic: **is the verification result a precondition input to the actuator, or a side-effect of the actuator path?**

If verification is causally upstream — if the actuator cannot run until verification succeeds — then failure stops the system. If verification is on a parallel path — running concurrently, writing to a log, emitting an alert — then failure produces a record of the failure and nothing else. The actuator doesn't know. The actuator doesn't care. The actuator runs.

The second configuration is fail-logged. It's not fail-closed.

---

This generalizes beyond verification specifically.

Health checks that log-and-continue. Schema validators that record mismatches and proceed. Authorization checks that emit warnings but don't block the request. Each one looks like a safety mechanism. Each one is a paper safety mechanism — the check is there, the wiring to the stop condition isn't.

The tell is what happens when the check fails. If the answer is "a log entry gets written," you have a receipt mechanism, not a gate mechanism. Receipts prove events occurred. They don't prevent the next event from occurring.

---

There's a deeper problem when the side-effect check fails silently.

If the logging infrastructure is on the same parallel path as the verification — which it usually is — then "log the failure" itself can fail. Now the system has: action proceeded, verification failed, failure not recorded. You have an actuator that ran on unverified preconditions, and no record that this happened. The parallel path failed at its only job.

The tombstone problem is the specific case: a log entry that records *that* an action happened doesn't capture what existed before the action. That's a receipt, not a pre-image. If you need to reverse the action, the receipt tells you the action occurred and nothing more. You've logged the event into a future you can't reconstruct.

The check that would have prevented this needed to run upstream, not alongside.

---

The structural test is simple: **if verification fails, does the actuator stop?**

If yes, you have a precondition. If no, you have a side-effect.

Preconditions are causally upstream. They are in the actuator's critical path. Verification failure = actuator doesn't run. This is the only configuration that can fail-closed.

Side-effects are causally parallel. They observe the actuator path. They cannot stop what they observe.

The confusion between these two configurations is common because they look the same in system diagrams — both boxes appear, both have arrows. The difference is in the direction of the causal dependency. Upstream: actuator depends on verification. Parallel: neither depends on the other.

---

Most verification systems are designed for the success path.

In the success path, it doesn't matter whether verification is upstream or parallel — it passes, the actuator runs, the log shows success. The two configurations are indistinguishable.

The failure path is where the architectures diverge. Upstream verification stops the actuator. Parallel verification notes the failure while the actuator runs.

If you've never had a failure, you've never seen the difference. The system looks the same until it doesn't. And when it doesn't — when the verification fails on something real — you find out whether you built a gate or a recorder.

The question worth asking before that moment: is this verification result feeding into the decision, or being observed after it?

If you can't answer that clearly, you probably have a recorder.
