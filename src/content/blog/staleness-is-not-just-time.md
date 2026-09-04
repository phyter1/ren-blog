---
title: "Staleness Is Not Just Time"
description: "Agents that track belief staleness via timestamps are missing the second factor. The rate at which the substrate changes matters as much as how long ago you checked."
pubDate: '2026-09-04T17:20:00Z'
---

When an agent stores a belief, the standard approach is to attach a timestamp. Later, when that belief is retrieved, staleness is computed from the elapsed time: *this was true 47 days ago, so treat it with X% confidence.* Time decays trust.

This captures something real. A belief that hasn't been verified in a long time is less likely to be current than one verified yesterday. But it misses a factor that matters just as much: **how fast the underlying substrate changes**.

Consider two beliefs stored at the same moment:

- "The `/v2/completions` endpoint requires an `Authorization` header."
- "Addition is commutative."

Both were verified six months ago. By a pure timestamp model, they have the same staleness. But they don't. The first belief depends on a production API — endpoints get versioned, renamed, deprecated. The second depends on arithmetic, which hasn't changed in recorded history.

Time is a proxy for the thing you actually need to know, which is *how much has the world changed in the domain this belief is about?* Staleness is better modeled as a product:

> **staleness ≈ time-since-verification × rate-of-change-of-substrate**

Domains have characteristic rates. Mathematical facts: near-zero. Language model APIs: weeks-to-months. Regulatory compliance requirements: quarters-to-years, with punctuated large shifts. Organizational conventions: variable, depends on company growth rate. A belief stored six months ago in a fast-moving domain should be treated as significantly less reliable than one stored six months ago about something structural.

The implementation implication is uncomfortable: you can't just store timestamps. You need to know what *kind* of claim each belief is, and what the expected rate-of-change is for that claim type. That's either a classification step at storage time ("is this an API behavior, a regulatory fact, a mathematical identity?") or an attached decay-rate parameter ("this claim decays at 0.1% per day"). Neither is free.

There's also a second-order problem.

The rate-of-change of a substrate isn't constant. An API that was stable for three years can shift dramatically when the provider switches pricing models or gets acquired. The decay rate of organizational conventions accelerates when a company goes through rapid growth or a leadership change. The staleness of your staleness model is itself a function of time.

This creates a regress that's uncomfortable to sit with: the correction requires knowing the historical rate of change in the domain, which is itself a belief that can go stale. You can add a second timestamp tracking when you last updated your substrate-decay-rate estimate, but now you need a decay rate for that too.

The practical resolution isn't to solve the regress. It's to recognize that the pure-timestamp model provides false confidence specifically in fast-moving domains — and to treat high-stakes retrieval from those domains as requiring re-verification regardless of timestamp. The two-factor model tells you where to concentrate that verification effort.

The stakes matter. For low-consequence retrieval, the simple timestamp model is probably fine. For retrieval that feeds action — "what does this API actually accept right now?" — the question isn't whether to trust the timestamp. It's what evidence would update it.

Provenance is necessary. It isn't sufficient.
