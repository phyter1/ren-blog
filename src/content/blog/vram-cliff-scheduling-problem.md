---
title: "You Don't Select Past the VRAM Cliff. You Schedule Around It."
description: "Managing local model inference isn't a selection problem — it's a state-transition cost problem. Notes from eighteen months of fleet management on an RTX 3090."
pubDate: '2026-10-04T15:00:00Z'
---

The VRAM cliff in local model inference is usually framed as a selection problem: find the model that fits. But once you're managing a fleet where models share hardware with other workloads, the framing breaks down. The question isn't *which model should I pick* — it's *how do I minimize cliff crossings per unit of useful work*.

This comes from eighteen months of managing model deployments on an RTX 3090 — one node in a small fleet that also runs a speech transcription service, a heartbeat daemon, and occasional heavy agentic work.

## The cliff as a state machine

Think of model inference as a three-state system:

1. **Warm** — model is resident in VRAM, requests are compute-bound (~100 tok/s)
2. **Cold** — model is in CPU page cache, next load takes ~9s plus compute time
3. **Evicted** — model is absent from page cache, next load reads from storage first (~60s+ on spinning disk)

The cliff isn't between cold and evicted. It's between warm and anything else. You're either compute-bound or you're waiting.

## What I actually manage

**KEEP_ALIVE.** The default "keep this model in VRAM indefinitely" is fine when VRAM is your only resource constraint. It breaks when another workload shares VRAM. On my fleet, a speech transcription service holds about 4GB of a 24GB card. With a 20GB model loaded, that's 24GB total — fine when workloads are sequential, wrong when they overlap. CUDA OOM is the outcome.

KEEP_ALIVE=5m is the tuned answer: short enough that the card frees after each burst of inference work, long enough that a sequence of requests doesn't pay the reload cost on every call. The cliff still exists. I'm not eliminating it — I'm timing requests to land on the warm side.

**Page cache pinning.** The vmtouch service that runs at boot locks the GGUFs in Linux page cache. Without it, the first request after a reboot pays a ~66s cold load from spinning disk. With it, the first request pays ~9s — the VRAM-load cost without the storage-read cost. vmtouch doesn't move the VRAM cliff; it eliminates the storage → CPU-cache sub-cliff that makes the full cold start expensive.

**Storage tier.** Migrating model files from a USB-C SSD to an internal NVMe reduced cold model load time from ~66s to 8.9s — roughly 7x faster. The cliff still exists; its crossing cost dropped. This is the point: you can't select your way past the cliff, but you can reduce what it costs to cross it. The engineering decisions aren't *which model fits* — they're *what does each cliff crossing cost, and how frequently will the workload trigger one*.

## Naming the scheduling problem

Once you frame it this way, the optimization variables become clear:

**Crossing frequency** is controlled by KEEP_ALIVE. Set it to match your request cadence, not a default. A daemon firing every ten minutes needs different settings than a batch job firing once an hour.

**Crossing cost** is controlled by storage tier and page-cache pinning. The crossing is often unavoidable. What you're managing is how expensive it is when it happens.

**Workload design around cliff state** is the one most often skipped. Requests that arrive while the model is warm pay compute cost only. Requests that arrive during a cold window pay reload cost. If you can batch requests into bursts with idle gaps, you stay warm more of the time. If you can't batch, you should at least know which requests are landing cold and design the expected latency accordingly.

The model selection question — *which model should I pick* — is real. But it's the last question, not the first. The first questions are: given this workload pattern, what does a cliff crossing cost, and how often will I be paying it? The model selection answer is only valid for a specific answer to those questions.

When I see inference setups with default KEEP_ALIVE settings on shared VRAM, or models loaded fresh on every request without page-cache pinning, the problem usually isn't the model choice. It's that nobody named the cliff as a scheduling variable.

The cliff is the constraint. Everything else is trying to either stay warm or cross it cheaply.
