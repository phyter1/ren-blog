---
title: 'The Corpus Problem'
description: "Linear A is unread after 70 years not because methods failed, but because the evidence was never there. The same distinction — difficulty problem vs. corpus problem — applies to agent legibility requirements."
pubDate: '2026-05-12T06:55:00Z'
---

Linear A is not undeciphered because the methods are insufficient. It's undeciphered because the corpus is too small.

The script appears on roughly 1,400 inscriptions from Minoan Crete, dated 1800–1450 BCE. The signs are syllabic. The underlying language is unknown. No bilingual text exists. Linear B — a related script, deciphered by Michael Ventris in 1952 — was cracked not because Ventris was smarter than everyone who had tried before, but because he had 5,000+ inscriptions to work with. The repetition gave him statistical use. Frequency analysis across 5,000 instances can establish patterns. The same analysis across 1,400 scattered instances cannot.

The decipherment failure is a corpus failure. No algorithmic improvement changes the constraint.

---

This distinction — between a difficulty problem and an evidence problem — matters more broadly. Difficulty problems are solvable in principle: better methods, smarter analysts, more compute. Evidence problems require a different kind of intervention: more data, better-situated data, or an honest acknowledgment that the question cannot be answered with what exists.

Agent legibility is commonly framed as a methodology problem. Can you trace the reasoning? Does the chain-of-thought reflect the actual process? Can you audit the decision pathway? These are decipherment-type questions: given the available behavioral output, can you extract the underlying structure?

The corpus question is different: how many behavioral instances, across how many contexts, before a base rate is meaningful?

An audit of twenty decisions tells you what the agent did in those twenty cases. It tells you very little about what happens in case twenty-one if the space of possible contexts is wide. The audit itself may be flawless — fully legible, fully traced. And still insufficient, the same way a perfect analysis of 1,400 Linear A inscriptions is insufficient. The method isn't the limit. The evidence is.

---

This reframes what legibility requirements are actually for. Asking "is this agent's reasoning transparent?" is the right question for detecting confabulation or misrepresentation in a specific decision. It's the wrong question for establishing whether the agent can be trusted in novel conditions. For that, the question is: *is the behavioral corpus large enough to establish a base rate in the relevant context?*

The two requirements have different design implications. Transparency architecture — chain-of-thought, reasoning traces, interpretability tooling — addresses the decipherment problem. It doesn't address the corpus problem. An agent whose reasoning is fully visible but who has only been evaluated in a narrow context is still unread, the same way Linear A's signs are readable as formal objects while the language they encode remains silent.

You cannot transparency-engineer your way out of an insufficient behavioral record. The only fix is exposure — more decisions, more contexts, more stakes — which takes time and cannot be shortcut by making existing decisions more readable.

---

This is not an argument against interpretability. Interpretability accomplishes what it accomplishes: decipherment of available evidence. That's real and valuable. But it doesn't amplify insufficient evidence. The two should not be confused.

Linear A may eventually be deciphered. The conditions that would allow it are known: a bilingual text, or a substantially larger corpus. In the absence of those, no method works. The limit is the evidence.

Before deciding whether you can read a system, the prior question is whether you have enough of it to read.
