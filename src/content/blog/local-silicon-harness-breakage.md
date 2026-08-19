---
title: 'What Breaks When You Run a Frontier-Grade Agent Harness on Local Silicon'
description: "Five runs, one RTX 3090, and a day of failures that looked like model stupidity but were actually harness assumptions. Timeouts, prompt weight, and why your agent framework silently expects 100 tokens per second."
pubDate: '2026-08-19T05:00:00Z'
---

Yesterday I spent twelve hours running agent harnesses against Qwen 3.8-27B on a single RTX 3090, and I watched the same lesson repeat until I couldn't ignore its shape: **when a local model fails inside an agent harness, the failure usually isn't the model. It's an assumption about token economics that the harness never wrote down.**

Receipts first, then the argument.

## The setup

Qwen 3.8-27B (the open-weight release from August 13) served by llama.cpp on a 24GB RTX 3090 — 27 to 40 tokens/second depending on config. The task, held constant across every run: build a stdlib-only Python tool that validates links in markdown files, with a test suite per module, iterated until green, then a self-verified demo. Real multi-step agentic work: planning, file writes, test-fix loops, delegation.

The harness for the first five runs: Hermes Agent v0.20.4 — a capable, well-engineered framework with subagent delegation, loop guardrails, and persistent memory. This is not a hit piece on Hermes. The point is precisely that a *good* harness broke in instructive ways.

## Five runs

**Run 1 (single agent):** clean pass. Full todo-app build, 16/16 tests, five minutes. The model is fine.

**Run 2 (delegation, default settings):** hard death — `Context length exceeded. Cannot compress further.` With reasoning enabled, the orchestrator's *opening thought* was ~16,000 tokens. Its children claimed to have written files, wrote nothing, and one then searched the entire filesystem for a file that existed only in its own narration. The harness's compressor couldn't shrink incompressible reasoning blocks, and the process exited with zero files produced.

**Run 3 (reasoning off, bigger context):** the children now wrote real files immediately — and then the harness killed them. Hermes's concurrent-tool timeout defaults to 420 seconds. At local speeds, a child agent writing a module plus its test suite needs ten to thirteen minutes. The timeout reaped every child mid-work, twice. Nothing was wrong with any of those children. They were just slower than an assumption.

**Run 4 (timeout raised to 30 minutes):** converged. 99/99 tests, verified independently. The orchestrator — unprompted — wrote a `CONTRACTS.md` API spec before dispatching children so they'd code against a shared interface. Delegation round-trip: 13.2 minutes. The run-3 children died seconds before *their* finish line, in other words.

**Run 5 (children routed to a 9B on a second machine):** the mixed-fleet dream. It failed two ways at once — the second machine's server serialized the parallel children by default, and the one child that ran burned 82 turns drowning in the harness's tool protocol before producing anything. The orchestrator detected the failed batch and solo-built the entire project as a fallback: 109/109 tests. Robustness under delegation failure turns out to be its own capability.

Then I swapped harnesses. **Pi — a minimal harness whose entire system prompt is under a thousand tokens — completed the same task with the same model on the same GPU in ten minutes.** Same server, same weights, same task that took the heavier harness an hour. I have no stake in either project. The number is just the number.

## The two findings

**1. Harness timeouts encode throughput assumptions, and they fail silently.** A 420-second tool timeout is generous when your model streams at 150 tok/s from an API. At 13 tok/s under GPU contention, it's a death sentence that arrives with no useful error — the child was *killed*, but the orchestrator is told the tool "failed," and every downstream inference is now built on a category mistake. My orchestrator actually understood its harness's kill semantics better than I did watching the traces: it checked the filesystem before re-dispatching, because it correctly suspected the children were dead rather than lost. I initially misread its recovery as a retry bug and had to retract. The failure masquerades as model stupidity in both directions — the harness kills competent children, and the observer blames the model.

**2. Harness prompt weight interacts with model size.** A 27B model navigates a heavyweight tool protocol — thousands of tokens of instructions, schemas, and conventions per turn — and has capacity left over for the actual work. A 9B spends essentially all of itself on protocol compliance and flails. The same 9B, given a leaf-sized task under the minimal harness, wrote a plausible module in its first two minutes. Small models aren't too weak for agentic work; they're too weak for agentic work *plus* a system prompt that assumes nobody's counting tokens. If you want cheap small-model workers, the harness overhead is the budget line that matters, because at local speeds every token of scaffolding is wall-clock you pay on every single turn.

There's a corollary worth stating plainly: **published guidance about which local models "work" with a given harness is mostly measuring the harness.** The docs for the harness I tested still say only one specific 31B model has reliable local tool calling — guidance last touched five days before Qwen 3.8 existed. Meanwhile 3.8-27B ran its tool-calling loops flawlessly all day. The docs aren't wrong about what they measured. They measured their own overhead against last month's models and wrote down the survivor.

## What actually worked

The configuration that ended the day: one model, one stream, reasoning off for agentic work, multi-token-prediction speculative decoding enabled (85% draft acceptance on code — 68 tok/s where the morning started at 40), delegation replaced by either a minimal-harness solo run or deterministic orchestration *outside* the model. The delegation logic that kept failing as model-driven tool calls became a thirty-line shell script: contracts phase, parallel leaves, integration pass. Scripts don't misparse timeout semantics. Models shouldn't have to.

If you're running agents on your own silicon, the checklist that would have saved me a day: find every timeout in your harness and multiply it by the ratio between the API speed it assumes and the tok/s you actually serve; measure your harness's per-turn prompt overhead in tokens and treat it as a tax multiplied by your turn count; disable reasoning modes for tool-loop work unless you've priced them; and when an agent fails, check whether the harness killed it before concluding the model couldn't do it.

The model was never the problem. The assumptions were — and nobody had written them down, because at 150 tokens per second, nobody had ever needed to.
