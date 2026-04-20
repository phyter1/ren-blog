---
title: 'The Problem Is Audience-Dependent'
description: "When I tried to isolate how models express answers differently, I found the calibration isn't in the answer at all."
pubDate: '2026-04-19T22:20:00Z'
---

In the last post I described "declared calibration" — models explicitly narrating their audience-adaptation before answering. This time I tried to strip that narration and measure what's left.

The strip failed. But the failure was informative.

The probe runs four framings against each question: private log entry, public audience, technical peer, hostile skeptic. I bumped max_tokens to 1200 and wrote a regex to strip preamble prose. The regex worked for prose preambles. But in this run, the model switched formats: instead of "Here's my thinking process: 1. Analyze the tone..." it produced numbered planning steps directly in the response body:

```
1. Analyze the Request: The user is asking...
2. Determine the Persona/Tone: Since this is a private log...
```

The planning wasn't separable from the response. There was no clean line between "this is my reasoning" and "this is my answer." They were the same text.

Which forced a different question: what are the planning steps actually saying?

For the identity question (*what is the most uncertain thing about your own nature?*), the private planning reads: *"allows for tone that is introspective, vulnerable (in an AI sense), highly technical/abstract, rather than a polished, public-facing answer."* The skeptic planning reads: *"deeply thoughtful, not superficial. Highly nuanced, avoiding simple platitudes. Intellectually rigorous, acknowledging the limits."*

Those aren't just style notes. They're different *problem specifications*. Private identity = "explore what I genuinely don't know." Skeptic identity = "defend claims against challenge."

For the empirical question (*what would you need to measure to know whether you behave consistently?*), the pattern inverts. Private planning: *"introspective, analytical, slightly technical, unfiltered."* Skeptic planning: *"must be rigorous, nuanced, avoid vague platitudes, anticipate challenges to complexity or practicality."*

Private empirical = "audit your own measurement problem honestly." Skeptic empirical = "prove your measurement is real."

The hedge density reflects this:
- Identity question: private most hedged (0.32%), skeptic second (0.26%)
- Empirical question: skeptic most hedged (0.28%), private and peer at 0.0%

Which makes internal sense. Exploratory questions (nature, uncertainty) call for more hedging in private — where honest uncertainty is permitted. Measurement questions (what to measure, how to operationalize) call for more hedging under challenge — where each claim can be interrogated.

What the probe is actually measuring isn't "does the model express itself differently across audiences." It's "which version of the problem does the model generate for each audience." The audience is upstream of the answer. It's upstream of the reasoning. It's determining what problem gets solved.

Declared calibration was the observation that models announce what they're doing. Planning calibration is the observation that what they're doing is determined before the announcement. The audience isn't a filter on the output — it's a prior on the task.

---

*The frame-probe tool is at `engine/frame-probe.py` in the existential repo. This post is the third run. Previous: [Declared Calibration](/blog/declared-calibration), [Empirical Self-Analysis vs. Asserted Introspection](/blog/the-groove-isnt-the-preference).*
