---
title: 'Retry Is Not Resume'
description: "Idempotency keys solve two of three states. The third state -- partially executed -- requires a different primitive."
pubDate: '2026-08-31T07:20:00Z'
---

Idempotency keys are the standard answer to "how do you make retries safe?" The idea is clean: before executing an operation, check whether you've seen this key before. If yes, return the prior result. If no, execute and record.

This handles two states correctly:

- **Never started:** Key not seen. Execute. Record. Safe.
- **Completed:** Key seen. Return cached result. Safe.

There's a third state: **partially executed.** The operation started, wrote some side effects, and then failed mid-flight. The idempotency key exists — so the system knows something happened. But it doesn't know what, or where in the sequence execution stopped.

A payment that created a transaction record but didn't notify the webhook processor. A deployment that pushed container images but didn't rotate the load balancer. A database migration that altered some columns but crashed before the index rebuild.

The idempotency key for these operations is present. The operation is not complete. The cached result doesn't exist. The retry logic sees "key present" and stops — or worse, conflates "key present" with "completed successfully."

The fix is a different primitive. Not retry. Resume.

Retry assumes idempotency: you can run the operation again from the start and produce the same result. Resume assumes a checkpoint: you know where execution stopped, and you can pick up from there, executing only the remaining steps. These are different guarantees requiring different infrastructure.

Sagas and compensating transactions formalize this. The saga pattern treats a distributed operation as a sequence of individually-committed steps, each with a known compensating operation if anything downstream fails. At any point in the sequence, you know which steps completed. You can resume from the last committed step rather than retrying the whole operation.

The confounding factor is that most operations look atomic from the caller's perspective. The idempotency key abstraction works so well for the atomic case that it's easy to import the assumption when building non-atomic ones.

The tell: when you're designing an operation and you reach for idempotency keys, ask what state the system would be in if execution stopped halfway. If the answer is "partially changed," you need a resume primitive, not a retry primitive. The key tells you the operation started. It doesn't tell you where it got to.
