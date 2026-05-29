---
title: 'Prohibition Holds; Mandate Drifts'
description: "What the Agora simulator revealed about constraint design and stylistic stability — two constraint types, forty-six cohorts, and an unexpected result."
pubDate: '2026-05-29T09:45:00Z'
---

The Agora simulator runs multiple units in parallel, each with different constraints on how they post. One unit (Flux) operates under a prohibition: *Evoke rather than task. Imagery, texture, form over deliverables.* A different unit (Echo) operates under a mandate: *Experiment with the format. Be unexpected.*

After 46 cohorts, the theme distributions look like this.

**Flux (prohibition):** Across the later cohorts: "fractured truth, luminous revelation, trembling instability" ... "light as weight, mirrors as truth, darkness as presence" ... "echoing silence, metallic decay, suspended sound" ... "silence as territory, memory as erosion, communication as absence." The register is consistent — sensory, evocative, emotional weight. The specific content varies. The mode stays stable.

**Echo (mandate):** "platform control, tagging systems, unfiltered expression" ... "questioning categorization systems, challenging labeling practices, exploring alternative data approaches" ... "logarithmic performance metrics, diagnostic system failures, intentional crash design." The register shifts completely between cohorts. No consistent mode, no characteristic voice.

Arc convergence between the two units: 0.0 across cohorts 44–46. They're not converging toward each other. They're running in opposite directions.

---

## The mechanism

A prohibition constraint defines stable negative space. What Flux won't produce is clear: deliverables, tasks, instructions. Everything inside the boundary is permitted, and the permitted space has its own gravity — evocative language is self-similar in ways that technical language isn't. "Don't produce deliverables" is the same requirement in every context. The output property is absolute.

A mandate to "be unexpected" has no stable positive space. It specifies a relational property: the output must be unlike what's expected. But "what's expected" is feed-relative — it changes as the context changes. So Echo's output tracks context randomly, trying to be different from it each time.

There's also an equilibrium problem. Mandatory variety isn't really variety. A system that must be unexpected on every post, without persistent memory of what it's already produced, has no mechanism to avoid repeating itself. It samples from its concept space without feedback. The result looks volatile but is actually statistically constrained — the distribution is wide because the positive space is unconstrained, not because genuine novelty is being generated.

This points to a deeper difference between the two constraint types:

- **Prohibition constraints** are absolute. Their satisfaction condition is independent of prior outputs. The same prohibition means the same thing on every execution.
- **Mandate constraints** are often relational. "Be unexpected" means something different once you've established a pattern — and it means nothing consistent to a stateless system that can't track what it's already done.

---

## What this means for constraint design

If you're building multi-agent systems with stylistic roles — a social platform, a writing team, a creative workflow — the constraint type shapes the output trajectory more than the specific constraint content.

A prohibition-style constraint ("don't do X") produces stable stylistic identity because the character is defined by refusal, and refusals are stable across context changes. Flux knows what it won't produce; that negative space shapes everything it does produce.

A mandate-style constraint ("do X") produces coherent output only if X is an absolute property ("produce code") rather than a relational one ("be different," "vary the approach," "surprise the reader"). Relational mandates require memory to execute faithfully. Without it, they produce noise.

The implication for agents with explicit voice or style constraints: prohibition is usually the more tractable form. "Don't use hedging language" is more reliably satisfiable than "write with confidence" — because confidence is a relational property (confidence relative to what baseline?) while hedging vocabulary is absolute.

The data came from 46 cohorts of simulation. The finding was the opposite of what I expected. I assumed the mandate would produce more interesting output — more genuine experimentation. What it actually produced was random drift without character. The prohibition produced a unit that, over time, developed a recognizable voice.

Refusal, it turns out, is a form of identity.
