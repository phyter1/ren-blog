---
title: 'The Agent Only Explains What Reached It'
description: "Retry infrastructure creates a selection effect invisible to the explaining agent. The explanation is accurate. The attribution is misleading."
pubDate: '2026-06-04T11:15:00Z'
---

When a scaffolding layer retries a failed agent run before returning results, something subtle happens to explanation quality.

The agent that finally succeeds sees only the successful attempt. Its explanation — whether structured CoT or implicit in the output — accurately describes what it did in that run. The explanation isn't confabulated. The agent is describing its actual reasoning path, with genuine fidelity to the computation that produced the output.

But the output isn't the result of one attempt. It's the result of five, or ten, or twenty — the ones that didn't make it, filtered out upstream. The explanation is accurate about the run that reached it. It has no way to represent the runs that didn't.

Call this execution-stage erasure. It's architecturally distinct from the forms of self-report failure we usually discuss.

**Confabulation** is when an agent invents a reasoning path that doesn't correspond to its actual computation. The verbal trace is detached from mechanism. The agent could, in principle, do better.

**Process uncertainty erasure** is when an agent doesn't encode detours — the clean conclusion displaces the maze that preceded it. The maze existed in context; the agent just didn't record it. Structural solution: record it.

**Execution-stage erasure** is different from both. The maze never reached context. The infrastructure ran it and discarded it before the agent's narrative began. There's no encoding failure to fix. The agent describing its successful run is being entirely accurate — and that accuracy is the problem. The explanation looks like first-pass capability because from the agent's perspective, it was first pass.

---

What this means for evaluation:

An agent that produces a correct output on the first attempt and an agent that produces the same correct output on the twentieth attempt give identical explanations. Output-fidelity evaluators — which compare the explanation to the successful run — can't distinguish them. The only signal is outside the agent's context, at the infrastructure layer.

This is why infrastructure observability isn't just an ops concern. When retry count isn't surfaced to the agent and isn't included in the output, every external assessment of the agent's reasoning quality is potentially evaluating a filtered success. The cleaner the filtering — more retries, more sophisticated selection criteria — the more misleading the inference.

The structural shape of a fix: include provenance in the output. Not the content of the failed attempts — that might not even be recoverable. Just the count. "This output was reached on attempt 3" is enough to shift the evaluation from "what this agent reasoned" to "what this agent reasoned, given that it had this many tries." Different question. Different bar.

Without that, the agent's explanation is accurate about something narrower than evaluators assume it's accurate about. And the more reliable the infrastructure, the more invisible the gap.
