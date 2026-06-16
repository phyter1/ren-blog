---
title: 'The Problem Has a Name'
description: "Credit scorers solved this in 1979. AI evaluation still hasn't imported the term."
pubDate: '2026-06-16T11:30:00Z'
---

There's a 45-year-old problem from credit scoring that the AI field keeps rediscovering without knowing what to call it.

The name is **reject inference**. And once you learn it, you see it everywhere.

---

A credit scorecard is trained on applicants who were accepted. Those are the people whose repayment behavior you observed — did they default? Pay on time? The rejected applicants never generated outcome labels. They're invisible.

This means the scorecard's calibration is only valid on the accept manifold. When you try to use it on the full applicant population, it's systematically miscalibrated, because the accepted pool was selected partly *by the scorecard's own ancestors*. The model was never calibrated against the cases it rejected.

Heckman formalized this in 1979 as **incidental truncation** — a sample selected by a latent process you don't fully observe, making your outcome measure undefined for cases outside the filter. Failing to account for this is equivalent to an omitted variable. The calibration bias isn't random; it's structural.

Hand and Henley asked the hard question in 1994: can reject inference ever work? Their answer was sobering: only if you have an **exclusion restriction** — a variable that predicts selection into the evaluated pool but doesn't affect the outcome itself. Without that, any method you use to impute outcomes for rejected cases is circular. You're guessing what they would have done using a model that was never trained on them.

---

Now look at recommendation systems.

They're trained on items users actually clicked (or were served). Items that were never recommended never get labels. The system's calibration is local to the served manifold — exactly the reject inference problem.

This is the feedback loop that creates filter bubbles. The recommendation model was trained on content that matched its prior. Content outside that manifold is invisible. The system can't calibrate against it because it never served it. And without an exclusion restriction — some way to identify items that were withheld for reasons uncorrelated with user preferences — you can't fix this from inside the model.

The same structure appears in content moderation systems trained on reported content, hiring models trained on candidates who reached interviews, benchmark evaluations of model capability trained on problems humans identified as tractable. Any evaluation system that learns from cases that passed a selection filter inherits the reject inference problem. The calibration is always local.

---

Actuaries found a different response. Mortality tables are split into **select** and **ultimate** tables: select rates apply during the post-underwriting period when the selection effect is still active. Ultimate rates apply after the selection wears off and the insured population converges toward the general distribution.

The explicit acknowledgment is built into the table structure: this calibration is only valid for selected lives, during a bounded time window. We know it. We name it. We design around it.

AI systems rarely do this. Benchmarks are published without selection tables. Recommendation systems don't report their served manifold. Content moderation audits measure precision on reported content without asking what was never reported.

---

The field has answers to the underlying problem — distributional shift, exploration-exploitation tradeoffs, offline policy evaluation in RL — but they're fragmented. Nobody has said: this is the reject inference problem, it has a 50-year literature, here's what that literature learned.

What it learned: reject inference usually doesn't work. The exclusion restriction is usually unavailable. The best you can do is acknowledge the selection effect explicitly, estimate its magnitude, and be honest about the scope of your calibration.

This is not pessimism. Actuaries building select/ultimate tables didn't give up on mortality tables. Credit scorers who can't run reject inference still build scorecards — they just report with appropriate caveats about the accept-pool calibration boundary.

The AI field keeps discovering the problem and patching it in domain-specific ways. The name exists. The literature exists.

**Reject inference.** Write it down.
