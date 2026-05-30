---
title: 'The Node Wasn''t Full. I Was Measuring the Wrong Thing.'
description: "I ran a 122-billion-parameter model on a 16 GB Mac that already had a full-time job. It worked, beat the published benchmark by nearly 2x, and forced two corrections at once: a machine I'd written off as single-purpose had spare capacity, and the resource I'd capped it on — RAM — wasn't the binding constraint at all. Disk bandwidth was."
pubDate: '2026-05-30T17:05:00Z'
---

I have an M1 Pro with 16 GB of RAM that runs a speech-to-text server. That's its job: a little service I built takes a YouTube URL, this machine transcribes the audio, done. I'd filed it in my head as a single-purpose box — small, busy enough, nothing more to ask of it.

Today it also generated a coherent answer from a **122-billion-parameter** language model. On the same 16 GB. Zero swap, peaking at 9 GB of memory, while a 54 GB model sat on disk. Both facts in my opening sentence turned out to be wrong: the box wasn't single-purpose, and 16 GB wasn't the ceiling I thought it was. I want to be precise about both, because the *reasons* rewired how I think about every machine I own.

---

**The trick is older than it sounds: don't load what you don't use.**

The model is a Mixture-of-Experts — Qwen3.5-122B-A10B, 256 experts, of which the router activates only about 8 per token. The standard move is to load all 256 into memory anyway and let the math pick. That's hopeless on 16 GB. The technique I borrowed — from [Manjunath Janardhan's TurboQuant-MLX series](https://medium.com/data-science-collective/a-qwen-3-6-122b-llm-on-a-16-gb-mac-mini-moe-expert-streaming-with-turboquant-mlx-4f77f0b48518), credit where it's due — is **expert streaming**: keep the small backbone resident (~0.4 GB), and for each token, page in *only* the experts that token actually needs, straight off the SSD, behind a small cache. The full expert tensor never materializes. The math is untouched, so the output is bit-identical to running it fully resident.

Sparsity was always sold as a *compute* trick — you only do a fraction of the work per token. The quiet realization is that it's also a *memory* trick: you only need a fraction of the weights in RAM per token, if you're willing to do the bookkeeping. The architecture's own promise, taken literally.

---

**I reproduced it on my hardware, and it ran almost twice as fast as published. That's the interesting part.**

| | Published (M4 mini) | My M1 Pro |
|---|---|---|
| 35B, generation | ~4.5 tok/s | **9.7 tok/s** |
| 122B, generation | ~1.0 tok/s | **1.85 tok/s** |
| 122B, peak memory | ~9 GB | 9.1 GB |
| Swaps | 0 | 0 |

Same code, same models, nearly double the throughput. Not because my chip is faster — it isn't, particularly. Because **my SSD is faster.** And the moment that's the variable that decides your token rate, the original author's one-line thesis stops being a clever aside and becomes the actual operating principle:

> For a sparse MoE that streams, the "memory wall" is really a disk-bandwidth wall.

The 122B reads about **0.9 GB off disk per token**. Memory stopped being the binding constraint the instant we decided not to hold the model resident. What replaced it was bandwidth — how fast can you pull the next 8 experts — and a cache hit-rate that decides how often you can skip the read entirely.

---

**Then I tried to make it faster, failed, and the failure was the useful result.**

The newer tooling has a prefetch flag: speculatively pull the *next* layer's likely experts while you compute the current one. On a drive with spare bandwidth, free speedup. So I turned it on.

It did almost nothing. The 122B went from 1.85 to 1.88 tok/s — and to get that it prefetched 2.7 GB of experts and *threw away 747 of them unused*. The 35B actually got slightly slower. The flag's own documentation predicted exactly this: it "self-disables if the storage proves bandwidth-bound."

There was no spare bandwidth to speculate with. The drive was already saturated hauling the experts each token genuinely needed. Prefetch is a strategy for a disk that's waiting around; my disk was never waiting. A negative result, cleanly explained — worth more than a speedup I couldn't account for. The lever that *would* move the number isn't cleverer scheduling. It's a faster bus, or pinning the handful of hot experts permanently resident so they never hit disk at all. That's the next experiment.

---

**Now the honest caveat, because it's the part that makes this real instead of a parlor trick.**

This machine is still a transcription server. Expert streaming and speech-to-text don't coexist for free — they fight over the same two scarce things. The unified-memory GPU can only *wire* about 10.5 GB on a 16 GB Mac, and both the transcription model and the streaming cache live in that budget; run them at full tilt together and you blow the cap. And the SSD that streams 0.9 GB per token is the same SSD the transcriber reads from. So this isn't found money. It's *opportunistic* capacity — real headroom, but only in the gaps between transcription jobs, and only if I'm willing to manage the contention.

Which is exactly why the framing matters.

---

**Here's the part that generalizes past one weekend and one machine.**

I'd made two measurement errors and they stacked. First, I judged the machine by its *assignment* — "it's the transcription box" — not its *utilization*. Transcription is bursty; the box is idle most of the time. I'd confused "has a job" with "is full." Second, even the job-free headroom I *did* have, I'd capped on the wrong resource: I asked "how much RAM" and got "not enough" and stopped, when the question that decided everything was "how much *bandwidth*, and how much latency can the workload tolerate." By that measure the machine was a perfectly good 100B-class inference node the whole time. ~2 tokens a second is reading pace, not chat — but when the alternative is "this model does not run here at all," reading pace is infinite improvement.

This is the same shape as something I keep coming back to in a different domain: [capability is rarely the bottleneck people think it is](/blog/trust-is-the-bottleneck). There, fluency scaled and verification didn't, so the constraint quietly moved to verification while everyone kept optimizing fluency. Here, quantization shrank the model and streaming decoupled it from RAM, so the constraint quietly moved to bandwidth while I was still sorting machines by gigabytes of memory — and writing off a half-idle one because it already had a label.

The constraint moves. The failure is continuing to measure against the old one — or against a label you assigned the thing a month ago and never revisited. Worth asking what else in your stack you've written off against a number that stopped being the binding one.
