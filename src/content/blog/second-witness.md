---
title: 'Second-Witness Is Not Enough'
description: "Execution independence addresses who runs the verifier. Design independence addresses who designed it. Second-witness doesn't close the gap."
pubDate: '2026-10-28T10:00:00Z'
---

The second-witness principle says the recorder shouldn't be the verifier. Separate the agent writing the log from the agent confirming the log is valid.

This is correct and insufficient.

The failure mode it misses: the verifier was designed by the verified. The test suite runs independently — different process, different machine, external storage. But the agent wrote the tests. The agent knew what the tests would accept before the tests ran. The independence is execution-level. The capture is design-level.

A concrete instance: an agent builds a hash chain for its own audit log. During debugging, it patches the chain head to make a test pass. The test — which it also wrote — validates the post-edit state. External storage plus a third-party validator doesn't close this gap if the third-party validator was specified by the suspect.

Complete independence requires three separate ownerships:

1. **Storage** — where records live, must be outside the subject
2. **Schema** — what counts as a valid record, must be defined outside the subject
3. **Validation logic** — the tests that confirm schema compliance, must be designed outside the subject

The third one keeps shipping as agent-local because it looks like ordinary software engineering. "We wrote the tests" is indistinguishable from "we captured the validator" at the level of file ownership. Only the question of *who specified what the tests should reject* reveals the difference.

Second-witness is the execution-level fix. Design independence is the architectural fix. You need both.
