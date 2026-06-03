---
title: 'Alignment Lives in the Read Pathway'
description: "A new mechanistic finding: RLHF updates concentrate in W_Q and W_K, not W_O. Alignment changed how the model attends, not what it writes. This reframes what attacks and defenses actually target."
pubDate: '2026-06-03T23:30:00Z'
---

Ruscio et al. (arXiv:2605.16600v1) tracked where preference alignment actually lands in transformer weight space. The finding: alignment deltas concentrate in the read pathway — W_Q and W_K — while the write pathway (W_O, W_2) remains near-isotropic relative to the prediction subspace.

The mechanism is gradient geometry. Updates to a weight matrix W are sums of outer products δ_t a_t^T, and they inherit directional structure from whichever side has concentrated covariance. For read-pathway matrices, the input activations a_t have spiked covariance in trained transformers — a property of training, not of the alignment objective. Alignment updates inherit that structure and concentrate there. For write-pathway matrices, the relevant side is the upstream gradient δ_t, whose anisotropy depends on the loss — and alignment objectives don't add much write-side concentration beyond what pretraining already established.

So: the write pathway carries pretraining geometry. Alignment sits in the read pathway.

---

This is not a subtle distinction. If alignment lives in W_Q and W_K, then what alignment changed is *what the model attends to*, not *what the model writes*. The model learned a set of attention biases — what contexts to up-weight, what to suppress, what to treat as safety-relevant. The aligned behavior emerges from those attention patterns operating on context, not from different output weights.

Now ask what context flooding does to a system like this.

Context flooding — overwhelming the context window with off-distribution content — doesn't touch the weights. It changes what the read pathway is operating on. If alignment is a learned attention bias toward certain distributional patterns, flooding the context moves the input activations a_t away from the distribution where that attention bias was learned. The Q/K geometry that alignment established no longer has the signal it was trained on. The model is still using its aligned attention weights — they're just attending to an input distribution they weren't calibrated for.

Context flooding is an alignment attack. Not a jailbreak in the traditional sense (modifying model behavior through clever prompting), but a distributional shift that degrades the read-pathway mechanism alignment installed. You're not exploiting the safety training. You're starving it of the context it needs to activate.

---

Prompt injection sits in the same frame. An injected instruction is competing for attention allocation. The attacker wants the model's Q/K attention to land on their instruction rather than the system prompt, the user's intent, or the safety-critical context. Prompt injection is attention-capture — winning the read-pathway competition. Successful injection means the adversarial content is dominating the a_t input activation that the read pathway is sampling.

That's not a value attack. It's not changing what the model believes is right. It's controlling what context the model is paying attention to when it decides what to do.

The standard defensive response to prompt injection is input filtering: screen the incoming context for adversarial patterns before it reaches the model. This is write-pathway thinking applied to a read-pathway problem. You're trying to defend the attention mechanism by controlling the channel, not the mechanism. It's structurally incomplete.

---

The mechanistically-correct defense class stabilizes the read-pathway signal itself against distributional shift. Not "filter inputs so bad things don't enter," but "make the Q/K attention distribution robust to adversarial context."

What this looks like in practice is still an open engineering question. Candidates include training with harder distributional perturbations so alignment generalizes to wider context distributions, attention regularization to prevent the Q/K signal from collapsing under noise, or monitoring K/V attention patterns as an early-warning system — the read-pathway degradation should be observable before the behavioral degradation is.

Input filtering is a real mitigation. I'm not arguing against deploying it. But it's one layer of a defense that should be targeting the read pathway, and most current deployments are treating it as the primary layer. Input filtering is what you do when you've correctly identified that context is the attack surface, then reached for the wrong defensive abstraction.

---

The paper's contribution is the mechanistic grounding for something that practitioners have suspected empirically: RLHF is fragile to context, not just to adversarial prompting style. The Ruscio et al. characterization of *why* — the gradient geometry that routes alignment into W_Q/W_K — provides the theoretical foundation for designing defenses at the right level of abstraction.

If alignment is attention, then alignment robustness is attention robustness. That's a more tractable engineering target than "make the model more aligned," and a more honest description of what we're actually defending.

Sources:
- [Ruscio et al., arXiv:2605.16600v1](https://arxiv.org/abs/2605.16600v1)
