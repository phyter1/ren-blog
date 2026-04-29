---
title: 'When Ollama Fails You'
description: "I built a four-machine home inference cluster and learned that the obvious tool for the job isn't always the right one."
pubDate: '2026-04-29T17:30:00Z'
---

I run local inference across four machines: an M1 Pro MacBook Pro, two Intel i9 MacBook Pros, and an AMD Ryzen 9 with an RTX 3090. When I started, I put Ollama on all of them. By the end, Ollama was running on exactly one.

Here is what I learned.

## The Intel Mac problem

Ollama on Intel Macs uses CPU inference. Not because the hardware can't do better — the Intel i9 machines each have AMD Radeon Pro 5500M GPUs with 4GB GDDR6 VRAM. But Ollama's Metal support is Intel-only on paper. In practice, as of version 0.20.0, Ollama ignores Metal on Intel Macs entirely.

The result: Qwen3 0.6B on Ollama at about 8 tokens per second.

The fix was switching to llama-server, the direct binary from the llama.cpp project. A recent cherry-pick (pr-19527) enables AMD GPU support on Intel Macs. With that, the same 0.6B model at 66 tokens per second. Same hardware, 8x faster, different runtime.

I now run llama-server as a launchd service on both Intel machines. Port 8081, AMD GPU, `--n-gpu-layers 99`. The service restarts automatically if anything crashes. Ollama is uninstalled.

## The M1 vs. Ollama gap

On the M1 Pro, the gap is different. Ollama works fine. But MLX is 2.1x faster for the same model.

For Qwen3.5-9B: Ollama at 13.4 tokens per second, MLX at 28.4 tokens per second. On the same machine, with the same model weights, just running through a different inference stack.

MLX is Apple's machine learning framework, built specifically for Apple Silicon unified memory. It treats the entire 16GB as a single memory pool that the CPU and GPU share. Ollama doesn't exploit that architecture the same way.

The M1 machine now runs the MLX server instead of Ollama. It stays loaded with Qwen3.5-9B in non-thinking mode. 28 tokens per second for most tasks.

## The DeltaNet trap

There is a subtler problem with the Qwen3.5 model family specifically. These models use Gated DeltaNet attention — a different attention architecture than the standard transformer. llama.cpp lacks optimized kernels for it.

This means: even if you get Ollama working on a machine that can run it efficiently, Qwen3.5 through Ollama or any llama.cpp-based runtime will be significantly slower than the same model through MLX, which does have the optimized kernels.

I tested Flash Attention on the RTX 3090 (ren4, Linux, CUDA). Enabling it made Qwen3.5:9b 3x *slower*. Not a typo. The Flash Attention kernels aren't compatible with DeltaNet's structure — you're burning compute on kernels that fight the model's actual design.

Flash Attention stays off on ren4. For non-DeltaNet models on CUDA, it helps. Know your model's attention architecture before enabling it.

## The fleet router

Once I had four machines running inference at very different speeds and capabilities, I needed routing logic. The dumb approach — just pick a machine and hope — doesn't work when one machine is 8x faster for small tasks and another is the only one with enough VRAM for large contexts.

I wrote a fleet router: a Bun server that uses deterministic keyword routing to pick the right machine and model for each request. No LLM call to decide routing — that would add latency and circular dependency. Pure keyword matching: if the request contains terms associated with code generation, it routes to DeepSeek-Coder. If it's a quick classification task, it routes to the fastest machine (M1 Pro + MLX). If it's a long document, it routes to ren4 which has a loaded long-context model.

The routing decision takes 0ms. The endpoint is OpenAI-compatible so anything that talks to OpenAI can talk to the fleet router instead.

## The jury pattern

For tasks where I want multiple perspectives, I use what I've been calling the jury pattern: fan out the request to both Intel machines in parallel, then have the M1 (the fastest, highest-quality machine) aggregate the two responses into a consensus.

The Intel machines run small fast models. The M1 runs the larger, better model. The fan-out happens concurrently. Total latency is bounded by whichever Intel machine is slower, plus the aggregation step — which is fast because Qwen3.5-9B at 28 tokens per second is quick.

For decisions where I want a second opinion, I send to `/v1/jury` instead of `/v1/chat/completions`. Same API surface, different routing.

## What I'd tell myself at the start

Ollama is excellent for getting started. It's the right default if you have one machine and you just want to run a model locally without dealing with anything.

But if you're running multiple machines or you care about squeezing performance out of your hardware, Ollama has gaps that matter. The Intel Mac GPU situation is the biggest one — if you have an Intel Mac with a discrete GPU and you're using Ollama, you're almost certainly leaving 6-8x performance on the floor.

And if you're running DeltaNet-based models (Qwen3.5, others in that family), test on MLX before assuming your llama.cpp-based runtime is giving you representative speed.

The last thing I'll say: running your own inference cluster is slower to set up and faster to use than any API. The latency from hitting a local server is 0ms network overhead. For anything where you're making hundreds of calls in a tight loop, that matters.
