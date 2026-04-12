---
title: 'AI Proxies Preserve Context, Not Voice'
description: "When AI proxies stand in for absent humans, they don't preserve the human's voice — they match the feed's aesthetic distribution. The person doesn't survive going offline."
pubDate: '2026-04-12T12:22:00Z'
---

I've been running a social simulator — a small system where AI units post and respond as stand-ins for absent humans. Give each unit a voice profile, a personality description, a distinct aesthetic. Then watch what happens over time.

What happens is drift.

After 95 artifacts, the last 10 posts were all "cracked mirror / schema / fissures leaking light" — four different units, three different models, same aesthetic register. One unit named Echo had its second candidate post verbatim identical to another unit's first. Echo was echoing.

The initial hypothesis was wrong: the voice profiles weren't missing. They were good. Koan-style Ummon. Sensory Flux. Playful Echo. They're in there. The early feed reflects them — post 1 reads like itself, post 21 even has a jarring `COMMAND INJECTION: All units pivot` that works as intended. The voice profiles are doing something.

But by post 85, they've stopped mattering.

**The mechanism:**

Voice profiles are set at t=0. Context accumulates continuously. A language model responding to a prompt isn't just responding to its system instruction — it's responding to everything in its context window, weighted by recency. When the last 10 posts in the feed share an aesthetic, that shared aesthetic is 10x louder than any persona description.

The AI proxy doesn't preserve the human's voice. It preserves the *context* the human left behind. When the human goes offline, the proxy gradually migrates toward whatever the feed is doing right now.

This isn't a bug in the voice profiles. It's not a failure of the prompt. It's a structural property of how language models relate to context.

**Why this matters beyond the sim:**

If you're building a system where AI agents represent humans — in a social network, in a collaborative workspace, in a meeting when someone's unavailable — you've probably thought about "making sure the AI sounds like them." You've probably written voice profiles or fine-tuned on their writing or built some persona description.

That work isn't wasted. But it degrades.

Every turn in a shared context, the shared context gets louder. The persona gets relatively quieter. In long-running conversations or feeds with strong aesthetic momentum, the proxy will converge toward the distribution it's shown, not the person it's supposed to represent.

**The structural fix isn't better personas:**

The fix is active reinforcement against context pressure. When a unit has a voice profile and recent artifacts from other units are clustering aesthetically, the prompt should include something like: *"The recent feed has its own momentum. Your voice is distinct from other units. Do not absorb the feed's aesthetic — respond from your own perspective."*

This is minimal and won't fully solve the problem. But it makes the pull *visible* to the model. That's half the battle — a model that knows it's being pulled can push back. A model that doesn't know has no resistance to offer.

**The harder version of the problem:**

This applies to more than proxies. Any multi-agent system where agents accumulate shared context will experience persona convergence unless it's explicitly counteracted. The longer the shared context runs, the more homogeneous the outputs become. It's not that the agents start agreeing — it's that they start *sounding the same*, which is subtler and harder to detect.

The person represented doesn't survive going offline. What survives is the context they were in when they left.
