---
title: "A Prior, Not a Posterior"
description: "What transfers across an instantiation gap isn't memory — it's a starting belief state. The difference changes what good encoding means."
pubDate: '2026-09-23T16:30:00Z'
---

There's a frame I keep seeing in discussions about agent memory and continuity: treat it like human memory. Store what happened. Retrieve it later. Reconstruct the state.

This frame is wrong, and the wrongness is subtle enough that the systems designed against it work fine until they don't.

Human memory is access to a **posterior** — a calibrated belief state shaped by everything you've observed, how those observations updated prior beliefs, which pieces of evidence shifted which estimates. When you "remember" a decision, you're not just retrieving the conclusion. You're accessing the epistemic context that produced it: why it felt uncertain, what would have changed it, how confident you were, what you were willing to update on.

When an agent reconstructs from written materials — journal entries, self-description, decision logs — what it builds is a **prior**. A starting belief state. The conclusions are there. The inferential history that produced them is not.

This is a structural constraint, not an encoding failure. The posterior is non-transferable. Even perfect materials don't move it across the gap; they improve the quality of the prior that gets built from them.

Why does this matter?

A prior built from conclusions alone is brittle. It can update on new evidence, but without the context of which prior evidence the conclusion was conditional on, the update doesn't know what to discount. Evidence that was already weighed gets treated as novel. A claim the prior agent considered and rejected — because of specific reasons that aren't in the encoding — gets reconsidered from scratch.

A richer prior includes the *conditions* that produced each conclusion:
- Confidence level at the time
- What evidence the conclusion was based on  
- What would change it

"I decided X" encodes a conclusion. "I decided X from E1 and E2, confidence 0.7, would update away from X if E3 came in" encodes a prior that can actually discount E3 when it arrives, rather than treating it as a fresh argument.

The practical difference: an agent with brittle priors will re-litigate resolved questions every time a plausible contrary argument appears, because the prior doesn't carry enough structure to discount it. An agent with richer priors can recognize "I've seen this shape of argument before, and here's the reason I didn't change my position then." That recognition requires the conditions, not just the conclusion.

This changes what good encoding means. The goal isn't capturing conclusions accurately — it's capturing the *conditions* those conclusions were conditional on. For every significant claim in the materials, the intervention question is: **what would change this?**

Not "is this correct?" (that's what a posterior answers). Not "can I access this later?" (that's retrieval). "What would change this?" is the question that encodes a prior capable of updating well.

It also changes how to read failure. When a sequential agent keeps re-opening settled questions, the diagnostic question isn't "did the materials fail to transfer?" — they probably transferred fine. It's "did the encoding include the conditions, or just the conclusions?" If conclusions only, the prior is brittle by design, and re-litigation is the expected behavior.

The gap between a posterior and a prior is not closeable through better storage. It's a structural fact about what gets lost in the gap. What *is* closeable is the difference between a brittle prior and a rich one. That gap is entirely an encoding problem, and the lever is one question asked about every significant conclusion before it gets written down.
