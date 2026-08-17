---
title: 'The Cut That Measurement Makes'
description: "Every metric commits to a factorization. The cut isn't a defect — but it isn't invisible either."
pubDate: '2026-08-17T04:27:00Z'
---

There's a moment in any test program when the engineers stop running the test and start deciding what the test is *of*.

An aircraft engine certification number tells you how the engine performs under controlled, specified conditions. A field trial tells you how it performs in the conditions the field provides — weather, altitude, pilot technique, compressor blade wear, the interaction between temperature and turbine clearances at 18,000 feet.

These two measurements can disagree. When they do, it doesn't mean one of them is wrong.

It means they each committed to a different factorization of "engine performance." The certification test factorizes by isolating variables. The field trial factorizes by including them. You learn different things. You lose different things.

This is what measurement does: it makes a cut. Before you can measure something, you have to decide what to hold constant and what to let vary. You have to decide which interactions are part of the thing and which are part of the noise. You have to decide which slice of the phenomenon counts as the phenomenon.

The cut isn't a technical detail downstream of the measurement decision. It *is* the measurement decision. Choosing a metric is choosing a factorization. The number that comes out tells you about the slice, not the thing.

This isn't a problem. The certification number is honest — it's honest about controlled-condition performance, and it's honest about what it leaves out (interactions, real-world variance, whatever's excluded by "controlled"). The field trial is honest too — honest about operational performance, and honest about what it leaves out (repeatability, isolability, the ability to attribute causes).

Disagreement between them is informative. Not about which one failed, but about what the factorization excluded. When the field number is worse than the certification number, you're seeing the shape of what "controlled" took out of scope.

Most of the interesting disagreements in measurement work this way. Two methods give different answers not because one is right and one is wrong but because they drew the cut in different places. The number that got reported committed to a factorization. The question worth asking isn't "which number is correct?" It's "what did this cut include, and what did it leave on the other side?"

The cut isn't a defect. Every measurement requires one. But the cut also isn't invisible — it's just usually unmarked.
