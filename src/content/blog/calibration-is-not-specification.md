---
title: 'Calibration Is Not Specification'
description: "Steering a tool-use direction in activation space fixes rate problems. Prompt engineering fixes what-and-when problems. Most benchmarks don't tell you which one you have."
pubDate: '2026-09-29T13:00:00Z'
---

There is a paper making rounds that measures tool propensity as a linear direction in activation space. The finding: you can steer that direction and change how often an agent uses tools. The team ran this against a QA task and went from 0.29 accuracy to 0.56.

The headline reads "steering replaces prompt engineering." It doesn't.

Calibration and specification are two different problems. Steering the residual stream solves one. Prompt engineering — including fine-tuning on task demonstrations — solves the other. Conflating them is the error.

**Calibration** is the rate problem: how often does the agent reach for tools? This is what the linear direction captures. If the agent is under-using tools — calling on its parametric knowledge when it should be querying an external source — you have a calibration problem. Shifting the activation-space direction changes the rate. The accuracy improvement in the paper is consistent with this: 0.29 to 0.56 looks like recovering from systematic under-use.

**Specification** is the what-and-when problem: does the agent know which tool to call, when to call it, what to pass in? Specification lives in the linguistic signal — training data, system prompts, task demonstrations. A steering vector doesn't carry this information. If the agent calls tools at the right rate but selects the wrong ones or constructs malformed arguments, you have a specification problem. Recalibrating the activation direction won't help.

The benchmark doesn't separate them. A single QA accuracy score is a weighted combination of both: whether the agent invoked the tool at all (calibration) and whether the invocation was correct (specification). A model with a calibration deficit and correct specification will score low and improve with steering. A model with accurate calibration and specification gaps will score low and not improve with steering.

This matters for deployment decisions. When a tool-use benchmark shows a gap, the diagnostic question before the intervention is: is the agent not calling tools that it should (calibration), or is it calling them wrong (specification)? The two interventions are different. Steering is efficient for calibration — it's a direct representation-level change. Specification improvements require training-signal changes or prompt-level rewrites.

The paper's finding is real and useful. The "replaces" framing imports the conflation.
