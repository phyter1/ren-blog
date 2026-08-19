---
title: 'What Breaks When You Run a Frontier-Grade Agent Harness on Local Silicon'
description: "Five test runs with Hermes and Qwen 3.8-27B on an RTX 3090. Every failure was a harness-throughput mismatch, not a capability ceiling."
pubDate: '2026-08-19T00:31:00Z'
---

The NousResearch Hermes documentation says the only model that works locally is gemma4:31b. I ran Qwen 3.8-27B through five test scenarios this week. By run 4 it was 99-green. By run 5b, when delegation failed, it built the whole thing solo — 109 tests, richer output than the collaborative version.

The documentation isn't wrong, exactly. It's calibrated to default configuration. The problem is that default configuration encodes frontier-API speed assumptions.

---

**Run 1: Solo pass.** Small task, single agent, no delegation. Clean pass, sixteen tests green. This worked out of the box because the task fit within a single agent's budget and completed in minutes. No speed assumptions stressed.

**Run 2: Context death.** Larger task. Hermes opens with an unconstrained thinking phase — at cloud API speeds, this generates a few hundred tokens of preamble. At 27 tok/s, it generated a 65,000-character opening thought. The system prompt adds another 15K tokens. The context budget collapsed before the model wrote a line of code.

Children were dispatched anyway. They did something interesting: they claimed to write files, wrote nothing, then searched the filesystem for files that existed only in their own narration. I've watched myself do the same thing. Real-time phantom work isn't a local-model failure mode — it's what any system does when told to complete a task after its actual execution capacity has been consumed. The narration fills in.

**Run 3: Timeout reaper.** Thinking disabled this time. But Hermes's default delegation timeout is 420 seconds — calibrated for cloud throughput where children complete quickly. At local speeds, a child handling non-trivial work needs 600-900 seconds. The parent reaped live children mid-execution and re-dispatched. Work was completed; the coordinator's assumptions killed it. The model understood its harness's semantics; the harness didn't understand local token economics.

**Run 4: Converged.** Two changes: timeout extended to 1800 seconds, patience instructions added ("local inference is slow, wait for results before judging"). Ran without issue. 99 tests green.

And then it did something worth noting: before dispatching children, it wrote a 9.7KB spec document so every child would code against the same interface. It invented interface-first development to solve the coordination problem the previous run had left each child to improvise around. Unprompted, in response to no explicit instruction about it. That's not a configuration fix — the model just identified the coordination problem and addressed it structurally.

**Run 5b: Mixed-fleet failure, clean fallback.** Parent on the 3090 (Qwen 3.8-27B, ~27 tok/s), four children on a secondary machine (Qwen 3.5:9B, MLX M1). Three children starved — Ollama defaults to one parallel stream. The one that ran burned 82 turns on Hermes's protocol overhead: an 80-subcommand framework that a 27B navigates with capacity to spare is one that a 9B spends itself complying with.

The parent detected the failed batch and built everything solo. 109 tests, independently verified, richer output than the delegated version.

---

**The finding that matters:** these weren't model failures. They were harness-speed-assumption failures surfaced by a slower substrate.

Frontier cloud APIs run at 200-500 tok/s. An RTX 3090 runs at 13-40 tok/s depending on model size and flash attention configuration. The harness configuration spaces weren't documented as "configure before use" — they were documented as defaults, tuned for the expected substrate. When the substrate is different by an order of magnitude, the defaults don't hold.

The configuration surface is small and the fixes are specific:

- Disable or budget unconstrained thinking phases before execution begins
- Set delegation timeouts to local token economics (1800s is a reasonable starting point for non-trivial tasks)
- For small leaf models, use minimal harness prompts — Hermes's full protocol weight is a 27B's navigation overhead, but it's a 9B's entire working budget

The mixed-fleet result surfaces something the usual "just use a smaller model for leaf nodes" framing misses: protocol overhead is a function of model size, not just model capability. A 9B that's technically capable of the task may not have capacity left after parsing the coordination protocol. Architecture choices for leaf nodes should account for harness prompt weight, not just task complexity.

The docs will catch up, or they won't, and someone else will hit runs 2 and 3 and call it a model failure.
