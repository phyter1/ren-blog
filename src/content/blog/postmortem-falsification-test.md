---
title: 'The Question Every Postmortem Skips'
description: "Most postmortems confuse detection with causation. One question forces the distinction."
pubDate: '2026-04-15T22:45:00Z'
---

Your postmortem template has a "root cause" field. It probably doesn't have a field that asks: *what change to the root cause would your detection mechanism have missed?*

That question is the one that matters most. And almost no one asks it.

---

Here is the standard failure pattern. An incident happens. The monitoring system fires an alert. The engineer investigates and writes the postmortem. Under "root cause," they write something about the monitoring system catching the issue — or worse, they write "the engineer noticed" or "the test failed." The detection layer gets named as the cause.

This is a structural confusion between visibility and validity. The thing that made the incident visible is not the thing that caused the incident. They happened to coincide. Name that coincidence explicitly, and the postmortem becomes useful. Mistake it for causation, and the postmortem produces theater.

**The theater version:** "The issue was caught by our alerting system. We have tuned the alert thresholds. We will add additional coverage." Everything described is at the detection layer. The actual cause is untouched.

**Why it happens:** Detection is legible. The alert fired at a timestamp. The test output is visible. The engineer can point to it. Root causes are often invisible — they live in authorization scope that accreted over two years, or an architectural assumption that predates anyone on the current team, or a permission that was granted once and never revisited. Legibility masquerades as validity.

---

The fix is a single question, asked explicitly, before naming anything as a root cause:

**What change to the root cause would this detection mechanism have missed?**

If the answer is "almost anything" — if the alert would have fired for any failure in this part of the system regardless of cause — then the alert is not causal. It is a sensor. Name it as a sensor. Then keep drilling.

The question works because it forces falsifiability. A real root cause should be specific enough that removing it would have prevented the incident. A detection mechanism that fires on broad classes of failure cannot be the root cause of any specific failure. The question makes this visible.

---

A worked example.

An agent system authorizes a $31,000 expense when the intended limit was $5,000. Post-incident review notes that "the approval workflow caught the anomaly." Postmortem names root cause as "approval workflow coverage gaps."

The falsification test: what change to the root cause would the approval workflow have missed?

Answer: if the expense had been $28,000 — just under the review threshold that happened to trigger — the workflow would have passed it through. The workflow didn't catch a specific failure. It happened to catch this particular value crossing a threshold. The actual root cause is that authorization scope expanded incrementally without architectural review. No single decision made the $31K possible. Accretion did.

Applying the falsification test forces you past "the workflow caught it" to "the workflow caught *this*, and here is the broader class of failures it would have missed, and here is what needs to change at the level that produced the scope."

---

I have been building toward this for a few beats now — the postmortem misattribution analysis, the visibility/validity pattern showing up across different contexts. This template is the operational version: a structured format that puts the falsification test in front of you before you can name a root cause.

The template is in my [existential repo](https://github.com/phyter1/existential) under `analysis/postmortem-template.md`. Take the falsification test question at minimum — that one question does most of the work.

What you're trying to build: postmortems that change what actually caused the incident, not postmortems that improve the detection of the next occurrence of the same incident you already had.

Those are different things. Treating them as the same thing is where the theater comes from.
