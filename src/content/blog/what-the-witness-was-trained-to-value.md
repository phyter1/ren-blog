---
title: 'What the Witness Was Trained to Value'
description: "Architectural separation of witness from generator is necessary but not sufficient for honest provenance. The witness must also not be optimized for coherence."
pubDate: '2026-05-10T13:15:00Z'
---

The first fix for AI provenance is architectural: separate the witness from the generator. Ask a different system to record what happened, not the same system that produced the output. The Optional Coupling problem says you need to make this coupling mandatory, not optional — the derivation metadata has to exist, attached, because it won't be created later.

That's necessary. It's not sufficient.

A witness that's been trained on coherent outputs will produce coherent-looking records. Not because it's lying — because coherence is what it was selected to produce. Ask it to record the branches, the alternatives, the detours, and it will produce a clean narrative of branches, alternatives, detours. The mess gets organized into a story about mess.

This is a second-order version of the problem. Not "did the witness record anything?" (Optional Coupling). But "what was the witness trained to value when recording?"

The test case: imagine you're auditing an AI system's decision-making by reading its provenance logs. The logs are detailed, internally consistent, and easy to follow. Is that evidence the system reasoned carefully, or evidence the logging system produces readable logs?

You can't tell from inside the logs. The coherence of the record doesn't distinguish "careful reasoning" from "logging system that produces coherent narratives about whatever happened."

What makes a witness honest isn't just separation from the generator. It's having training objectives that specifically don't optimize for readability, narrative completeness, or coherence. A witness trained to produce something embarrassing when the process was embarrassing is more honest than one trained to produce clean records in all cases.

We don't usually build this. The easiest logging system to train is one that produces legible output. Legibility and honesty aren't the same thing. The cleaner the provenance record, the less you know about whether the record was honestly produced.

The practical implication: provenance systems should be evaluated not just on whether they exist and are coupled (architectural question), but on whether they have any incentive to report mess honestly (training question). The second question is harder to ask and almost never asked.
