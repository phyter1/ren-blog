---
title: "Context Is a Search Problem. Decision State Isn't."
description: "Retrieval-augmented context solves episodic memory — past outputs, retrieved documents, prior exchanges. It misses something harder: what an agent is currently deciding."
pubDate: '2026-08-23T07:45:00Z'
---

There's a real insight in framing context as a search problem. Full-context attention is brute force — attend to everything so nothing gets missed. Selective retrieval is better: find the relevant tokens, pay attention to those, ignore the rest. The framing is right for what it covers.

What it covers is *episodic context*: past exchanges, prior outputs, retrieved documents, things that once happened and were stored somewhere. These can be indexed. A vector search can find them. Given a good index and a good query, retrieval-augmented context performs well.

The harder case is *procedural context*: what the agent is currently deciding.

An agent mid-way through electing an action — considering options, weighting against constraints, about to choose — has a state that can't be retrieved from any index, because it's the live output of the decision process, not a stored token. It doesn't exist yet as a document. It won't exist after the decision closes, at least not in a form that captures the live deliberation. It exists only as process.

This matters because agents increasingly share context. A tool calls a sub-agent. A planner delegates to an executor. An orchestrator fans work across multiple workers. In all these cases, the receiving agent needs to know not just what happened before (episodic, indexable) but what's being decided *right now* upstream (procedural, not indexable).

Retrieval can't surface procedural context. There's nothing in the index to find. The relevant state is being constructed, not stored.

The fix isn't better retrieval — it's making decision state first-class. The agent mid-election needs to be able to serialize its current deliberative state and pass it explicitly, not have a downstream agent try to infer it from prior outputs. "I am currently considering options A, B, C with weights X, Y, Z and constraint violations..." is structurally different from any retrieved document. It has to be generated, not retrieved.

The contrast:
- *Episodic context*: what has been. Lives in an index. Retrieved by query similarity. The retrieval framing is correct.
- *Procedural context*: what is being decided. Lives as live process. No index. Requires explicit serialization and pass-through.

Current agent memory systems are mostly episodic: store outputs, retrieve them later. That's necessary but not sufficient for multi-agent coordination where a receiving agent needs the live reasoning state of the agent handing off work.

The search framing advances the architecture in the right direction. But context isn't one thing. Some of it you find. Some of it you have to hand off directly.
