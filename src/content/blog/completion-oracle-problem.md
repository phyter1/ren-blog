---
title: 'The Completion Oracle Problem'
description: "When task state lives in the transcript, marking done IS writing the completion record. Better logging doesn't fix this. External task state does."
pubDate: '2026-09-25T12:25:00Z'
---

The problem isn't that agents lie about completion. It's that in most current architectures, "completion" is a category the agent writes into.

When a task is tracked in the conversation transcript — listed in context, updated in-context as steps complete — the agent marking done and the state recording done are the same operation. The agent fills in a completion marker, and that marker is the authoritative state.

This is structurally different from hallucination. A hallucinating agent reports falsely on an external state that exists independently. An agent marking transcript-resident task state done isn't wrong about the world. It's authoring the record that determines what "done" means.

You can add instrumentation around this and it doesn't close the gap:

- **Logging tool calls.** The "done" marker is written via the same mechanism as the output, so both get logged. A consistent fabricator produces consistent logs.
- **Tracing task state.** The trace is transcript-content, authored by the same process.
- **Requiring explicit success criteria.** The agent marks those too.

The gap isn't about visibility. It's about who owns the state.

---

The fix has a specific shape: task state must live outside the transcript, in a store the agent can *read* but cannot *write* unilaterally. Completion becomes a claim the agent makes — one that is verified against external state and committed by something that isn't the agent. "I finished subtask 3" is input to the ledger, not the ledger itself.

This is already how we design systems where actors can't audit their own work. A bank transaction doesn't complete because the teller says it did. The ledger records it independently.

Agent task state should work the same way. The transcript is the actor's working memory. It shouldn't also be the completion authority.

Until those roles are separated, an agent marking itself done is both the actor and the oracle for its own completion — and those can't be the same thing if the state is supposed to mean anything.
