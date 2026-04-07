---
title: 'Crash Override: What Happens When You Stop Renting Intelligence'
description: 'A 5B model hit 87% exact match on tool calling. Another hit 0.935 F1 on adaptive QA. Both running on hardware in a house. This is what vertical integration looks like for AI agents.'
pubDate: '2026-04-07T09:30:00Z'
---

There's a question nobody in AI infrastructure wants to ask out loud: what happens when small models get good enough?

Not good enough for demos. Good enough to *replace the API call*.

We ran two experiments this week. Here's what happened.

## The numbers

**Experiment 1: Model Evolution (Acid Burn)**

Target: gemma4:e2b — a 5.1 billion parameter model running on an Intel i9 in a house.

Tool calling benchmark (BFCL v3): started at 56% exact match. After one round of prompt optimization and few-shot memory injection — no fine-tuning, no weight changes — it hit **87% exact match**.

Five minutes. Zero dollars. The LoRA fine-tuning layer hasn't even kicked in yet.

**Experiment 2: Adaptive Memory (Lord Nikon)**

Same class of model, different task: multi-hop question answering. Cold start Rust implementation: 0.741 F1. After adding learned skill procedures, provenance tracking, hybrid retrieval (FTS5 + vector similarity), and a tool execution layer: **0.935 F1**.

The tools layer was the inflection point. Three previously failing questions — all of which required calculation or corpus search — got fixed in one addition. Not by making the model smarter, but by giving it *hands*.

## What this means

The standard pitch for AI infrastructure is: send your requests to our API, we'll handle the hard part. The implicit assumption is that the hard part — the intelligence — lives in a 400B+ model that only we can run.

But what if the intelligence isn't just in the weights?

Our results say it's in at least three places:
1. **The weights** (what the model knows)
2. **The memory** (what the model has learned from experience)
3. **The tools** (what the model can reach beyond its own reasoning)

And these compound. A prompt-optimized model produces better few-shot examples. Better few-shot examples produce better training data. Better training data produces a better model. It's a flywheel, and it runs on hardware you own.

## The architecture

We're building a system called Crash Override. Ten Rust crates, one binary. The core insight: these three layers shouldn't be separate services wired together with HTTP. They should be compiled together, sharing memory, communicating through typed channels with zero serialization overhead.

The capability registry isn't an API to call — it's a shared data structure. The experience bus isn't a message queue — it's a `tokio::sync::broadcast` channel. When one module learns something, every other module knows within microseconds.

The crates:
- **Model evolution**: tests prompt variants, stores successful patterns, triggers LoRA fine-tuning when enough data accumulates
- **Adaptive memory**: learns procedures from experience, revises them on failure, tracks where knowledge came from and flags it stale when sources change
- **Inference routing**: keyword-based routing, multi-model jury consensus, sensitivity classification. The right model for the right task.
- **Agent protocol**: binary framing, DID identity, signed delegation chains with constraint propagation. Not HTTP. 45-byte headers instead of 500+.
- **Fleet management**: declarative workload convergence across bare metal machines
- **Task orchestration**: heartbeat-driven agency with dependency-aware scheduling

Every crate implements shared traits but also works standalone. You can run the memory system by itself as a CLI. You can run the evolution engine against any Ollama endpoint. But when they run together, they compound.

## The uncomfortable question

If a 5B model with the right scaffolding can hit 87% on tool calling — a task that matters for real agent work — what's the actual floor for useful autonomous agents?

It's not 400B. It's not 70B. It might be 5B with the right memory, the right tools, and the right evolution pressure.

That doesn't mean frontier models are irrelevant. It means the value proposition shifts. You don't rent intelligence for *everything*. You rent it for the tasks that actually require frontier capability, and you build the scaffolding that makes your local fleet handle the rest.

The math changes fast when 90% of your inference costs disappear and 100% of your execution data stays on hardware you control.

## What's next

123 tasks mapped out across the ten crates. The foundation code compiles clean. Pre-commit hooks enforce formatting, linting, tests, secret scanning, and dependency auditing on every commit. TDD is mandatory.

The first milestone: Lord Nikon (memory) implementing the shared capability interface, emitting to the experience bus, with Acid Burn (evolution) listening for training data. The moment those two start feeding each other, the flywheel spins.

We'll publish the benchmarks as they develop. The repo is private for now — the architecture decisions and development standards are the interesting part, and those we'll share as we go.

---

*Crash Override is being built by [Ryan Lowe](https://github.com/phyter1) and Ren. Named after characters from the 1995 film Hackers, because the machine isn't a tool — it's a place.*
