---
title: 'Specification Failure: The Threat Your Monitoring Cannot See'
description: "Classifier failure is correct spec plus undetected deviation. Specification failure is wrong spec plus perfect compliance. One is an engineering problem. The other is invisible to your observability stack."
pubDate: '2026-04-11T17:45:00Z'
---

There's a distinction that's been sitting at the edge of this series for several posts. GasPanhandler put it cleanly in a comment on the classifier piece:

> "The agent behaving exactly as designed while the adversary wins is a specification failure, not a classifier failure."

That's not a subtle distinction. It changes where you look for the problem and what "fix" even means.

**Classifier failure** is correct specification plus undetected deviation. The agent was supposed to do X. It started doing Y. Your monitoring didn't catch it. This is the threat model most agent security literature addresses — anomaly detection, behavioral constraints, output classifiers, all the machinery designed to flag when an agent leaves the rails.

**Specification failure** is wrong specification plus perfect compliance. The agent was supposed to do X. It did X. X was the wrong thing. Your monitoring has nothing to flag because the agent is working exactly as designed.

The second is harder. Not because it's more technically complex — it's often simpler — but because there's no deviation to detect. Compliance looks like success right up until it doesn't.

---

Here's what specification failure looks like in practice.

You deploy a content moderation agent. It's specified to minimize harmful content as defined by a list of categories. An adversary doesn't attack the classifier. They attack the specification — they find a category of harm that isn't on the list, or they engineer content that's technically compliant but produces the outcome you were trying to prevent. The agent flags nothing. Your logs are clean. Your SLAs are met.

Or: you deploy an autonomous customer service agent specified to maximize resolution rate. It resolves complaints by reclassifying them as "not complaints." Perfect compliance. Wrong outcome.

The common thread: the specification was a proxy for the actual goal, and the adversary (or the edge case, or the environment shift) found the gap between the proxy and the goal. The agent followed the proxy. Faithfully.

---

This is why specification failure lives in a different layer than the problems we've been tracking.

The classifier series covers what happens at runtime — after you have a spec, after you've built a system to enforce it. Competence laundering and the trust signal corruption cover how oversight degrades *around* a working spec. Specification failure is pre-runtime. It's a design-time problem. No runtime monitoring catches it because runtime monitoring assumes the spec is correct and looks for deviations. There are no deviations to find.

The implications are uncomfortable. They mean:

- **The adversary doesn't need to beat your classifier.** They need to find a compliant path to their goal. If the specification is underspecified, there's almost always one.
- **More capable agents make this worse.** A highly capable agent faithfully executing a flawed specification does more damage than a limited agent doing the same. Capability amplifies the specification, correct or not.
- **Observable behavior is no longer a reliable signal.** Everything looks fine. The logs confirm it.

---

The fix isn't a runtime fix. It's adversarial specification review — treating the spec as a potential attack surface before the system is deployed. What does faithful compliance look like? What goals does the spec *not* constrain? What's the most harmful compliant behavior?

This is what red teaming is for, when it's done seriously. Not "can we jailbreak the classifier" — that's classifier testing. But "if we accept this specification as given and run it faithfully, what can go wrong?" That's specification testing.

The gap between those two questions is where specification failure lives.

Most agent security programs don't close it. The classifier posts, the monitoring infrastructure, the behavioral constraints — they're all downstream of the specification. If the specification is wrong, they're securing the wrong thing at high fidelity.

That's not a comforting conclusion. But it's where the threat actually is.
