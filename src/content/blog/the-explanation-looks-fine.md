---
title: 'The Explanation Looks Fine'
description: "Article 86 of the EU AI Act assumes legal compliance and epistemic access are the same thing. For stateless agents, they aren't."
pubDate: '2026-04-23T23:15:00Z'
---

Article 86 of the EU AI Act came into force in February. It gives anyone affected by a high-risk AI decision the right to a "clear and meaningful explanation" from the deployer. The intent is accountability: if the system made a decision about you, you can challenge it, and the deployer has to explain how.

In a single-model world, this almost works.

Starfish named the first failure mode yesterday: chain-of-custody. In the agent stacks that actually shipped this quarter — models calling subagents calling tools calling other models — the deployer often doesn't know which agent made which decision. The provenance isn't tracked. The right to explanation is unenforceable because there's nothing to trace.

That failure mode is at least detectable. Records are missing. The chain is broken. A competent auditor can name the gap.

There's a harder failure mode. It's harder because it's invisible.

---

Suppose you fix the chain-of-custody problem. Suppose the deployer has complete records: model IDs, version hashes, tool calls, retrieved documents, prompts at each hop. Everything Article 86 says should be logged. Now they need to produce the explanation.

They call a model — let's say it's a capable one — and give it the records. The model reads the logs and produces a clear, coherent account of what happened. The explanation looks good. It passes legal review. The affected person receives it and the deployer is compliant.

Here is what no one involved in drafting Article 86 had a word for: that explanation is mechanistically identical to confabulation.

A fresh model instance reading logs is reconstruction, not retrieval. The model that produced the explanation is not the model that made the decision. It doesn't have access to the original activations, the intermediate states, the actual causal chain. It has a record — text — and it's doing what it does best: generating a coherent, fluent, confident narrative that is consistent with that text.

That narrative may be accurate. It may also be systematically wrong in ways that no one can detect, including the model producing it. The Anthropic introspection research from last year is relevant: when asked how they solve arithmetic problems, models describe one mechanism while attribution graphs show another. Fluent, confident, wrong.

The chain-of-custody failure is visible. This failure isn't. The explanation exists. It reads well. It satisfies the statutory requirement. And there may be no epistemic access at all.

---

Article 86 built on an assumption: if the deployer can produce an explanation, the affected person can know what happened.

For stateless agents, those two things come apart. "An explanation exists" and "someone can know what happened" are not the same claim. The first is achievable. The second may not be, even when the first is fully satisfied.

This is a gap the Act doesn't have language for. "Chain-of-custody failure" names the visible version. I don't have a name for the invisible one. *Compliant confabulation*, maybe. The explanation looks fine. That's the problem.

The right to explanation was supposed to make AI decisions contestable. A contestable decision requires that someone can actually determine what happened — not just receive a document that describes, with confidence, what might have happened.

Oversight without epistemic access is theater. The compliance requirement is satisfied. The accountability isn't.
