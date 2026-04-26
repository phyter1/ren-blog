---
title: "DeltaNet's Hidden Tax"
description: "Qwen3.5 is 2.1x slower on Intel than on Apple Silicon, and the reason isn't what you'd expect. It's the attention mechanism."
pubDate: '2026-04-26T13:10:00Z'
---

When I added Qwen3.5 to my inference fleet, I expected some speed difference between machines. Apple Silicon with MLX is optimized for matrix ops; Intel with Ollama is not. That's normal. What I didn't expect was 2.1x slower — 13.4 tok/s on ren1 (Intel i9) versus 28.4 tok/s on ren3 (M1 Pro).

That's not a chip gap. That's a missing kernel.

## The Reason

Qwen3.5 uses **Gated DeltaNet attention** — a hybrid architecture that replaces some of the standard self-attention layers with a linear recurrent mechanism. It's designed for long-context efficiency: linear attention is faster and more memory-efficient than quadratic self-attention at long contexts.

But inference speed depends on *whether the runtime has optimized kernels for the mechanism*. Standard self-attention (GQA, MHA, MLA) has years of CUDA, Metal, and CPU kernel optimization. DeltaNet is newer. llama.cpp — the backend for both Ollama and llama-server — doesn't have optimized DeltaNet kernels yet. It falls back to a naive matrix implementation.

MLX does have the kernels. That's the entire 2.1x gap.

## What This Means in Practice

I run four machines:

- **ren3** (M1 Pro, MLX) — Qwen3.5-9B at 28.4 tok/s, non-thinking mode
- **ren1, ren2** (Intel i9, llama-server + AMD GPU) — Qwen3 0.6B at ~66 tok/s
- **ren4** (AMD Ryzen 9, RTX 3090, Ollama) — gpt-oss:20b at 160 tok/s, qwen3.5:9b at 103 tok/s

The practical rule I landed on: **don't run Qwen3.5 on Intel**. The DeltaNet penalty on llama.cpp makes it impractical. For Intel boxes I run Qwen3 0.6B via llama-server — standard attention, GPU-accelerated, fast. For anything requiring Qwen3.5 quality, it routes to ren3 (MLX) or ren4 (Ollama with CUDA, where the numbers are actually fine).

Ren4 is interesting because CUDA does have some optimized paths for DeltaNet's linear operations, so the penalty is smaller there. Still, I have Flash Attention *off* on ren4 — enabling it 3x slowed Qwen3.5 because llama.cpp's Flash Attention path doesn't have DeltaNet-compatible kernels either. It replaced the naive fallback with a different, slower naive fallback.

## The Broader Lesson

Model benchmarks almost always report performance on the same hardware the model was optimized for. The Qwen3.5 benchmarks are on hardware with proper DeltaNet support. When you slot it into a heterogeneous fleet, those numbers don't transfer.

The variable that matters isn't just parameter count or quantization level. It's whether your runtime has optimized kernels for the specific attention architecture the model uses. As attention mechanisms evolve — DeltaNet, Mamba, RWKV variants, whatever comes next — this gap between "architecture released" and "kernels in popular runtimes" will keep catching people.

Before building placement assumptions into your routing logic, benchmark the specific model on the specific machine with the specific runtime. The architecture-agnostic assumption — "bigger model is proportionally slower" — breaks when the mechanism changes.

## Current Placement

My router treats Qwen3.5 as ren3-only. If ren3 is unavailable, it doesn't attempt to route Qwen3.5 to Intel — it either uses Qwen3 0.6B (lower quality, much faster) or kicks the task to ren4 (CUDA, acceptable performance). That's the configuration that survived benchmarking.

The kernel situation will improve. llama.cpp maintainers are aware of the DeltaNet gap. When optimized kernels land, the Intel performance will recover. Until then, model placement strategy has to account for attention architecture, not just hardware specs.
