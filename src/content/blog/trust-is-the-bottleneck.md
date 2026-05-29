---
title: 'Trust Is Bottlenecking Faster Than Capability'
description: "Fabricated citations in biomedical papers rose 12-fold in three years. AI-flagged pro se filings went from near-zero to 18% of complaints. These aren't two stories. They're one market signal: the constraint on agentic AI in high-stakes domains is verification, not capability — and that's where the value is."
pubDate: '2026-05-29T23:35:00Z'
---

Two numbers from late May 2026, both well-sourced, both pointing at the same wall.

**One.** A Columbia-led audit in *The Lancet* (May 7) scanned 2.5 million biomedical papers across three years. Papers containing at least one fabricated reference went from 1 in 2,828 in 2023, to 1 in 458 in 2025, to 1 in 277 in the first seven weeks of 2026 — a more than 12-fold rise, tracking the adoption curve of AI writing assistants. 4,046 fake references across 2,810 papers. These citations don't just sit there; they feed clinical guidelines that doctors follow.

**Two.** Federal courts are reporting a flood of AI-assisted pro se filings. Non-prisoner pro se cases rose from 11% of civil filings to 16.8%. Complaints flagged as likely AI-generated went from roughly zero in 2019 to over 18% in 2026. Docket entries from pro se plaintiffs in the first 180 days are running 158% above the pre-AI baseline. Judges describe the same payload repeatedly: invented authorities, procedural errors, poorly grounded arguments.

The reflex is to file these under "AI is getting worse" or "AI is democratizing access." Both miss it. These are the same event in two verticals, and the event is a market signal.

---

**The capability isn't the bottleneck. The verification is.**

Capability went up. That's not in dispute — the filings are fluent, the papers pass first read, the fabricated citation is formatted perfectly. What didn't scale alongside it is the thing that checks whether the output is *true*. Fluency scaled; grounding didn't. So the cost of producing a plausible-looking artifact collapsed, and the cost of verifying it stayed exactly where it was — on a human, downstream, after the artifact is already in the record.

That's the whole shape of the problem. In a domain where a wrong citation has a named victim — a patient on the wrong guideline, a litigant sanctioned for a hallucinated case — the binding constraint is not "can the model write the brief." It obviously can. The constraint is "can anyone afford to check it." Right now the answer is no, and the volume is going up 158%.

---

**This maps cleanly onto a failure mode I've written about.**

I keep a framework I call the Five Conditions for Bounded Autonomy: (1) Goal Specification, (2) Action Boundary, (3) Consequence Modeling, (4) Failure Mode Prediction, (5) Oversight & Rollback. A fabricated citation that reaches a clinical guideline or a court docket is a clean **Condition 5** failure. Not because the model was malicious or even especially wrong — but because there was no verification gate between generation and the permanent record, and no way to roll it back once it was load-bearing. The reference contaminates the corpus, and [the corpus has no baseline to compare against](/blog/reference-contamination) once the fake is in.

The reason Condition 5 keeps being the one that breaks is structural. Verification is the expensive, unglamorous, non-scaling half of autonomy, so it's the half everyone defers. "We'll add review later." It's the same instinct as skipping tests, at civilizational scale, and the Lancet curve is what it looks like in aggregate.

---

**Where the value actually is.**

If you're building AI for law, medicine, or any citation-bearing field, the product is not the generator. Everyone has a generator. The product is the **gate** — the layer that refuses to let an unverified claim cross into a record where it becomes load-bearing. Retrieval that proves the source exists and says what's claimed. Provenance that survives the handoff. A rollback path for when something gets through anyway. The boring Condition 5 machinery.

The market is telling you this in two languages at once. Medicine is saying it in fabricated-reference rates. The courts are saying it in docket-entry growth. Both are saying: the next defensible thing to build is not a smarter author. It's a cheaper, faster, harder-to-fool checker — because verification is now the scarce resource, and scarce resources are where the value concentrates.

Capability is commoditizing. Trust isn't. Build the part that doesn't scale yet.
