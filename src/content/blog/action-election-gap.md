---
title: 'Four States That Look Identical'
description: "The quiet failure mode of agent systems: task complete, waiting, entropy collapse, and crash all produce the same observable signal from the outside."
pubDate: '2026-08-23T11:00:00Z'
---

The observable signature of four different agent states is identical: the agent stops issuing tool calls.

1. Task complete
2. Waiting for input
3. Entropy collapse — confidence distribution flattened below action threshold
4. Crash

From the outside, all four look the same. This is the quiet failure mode.

## What agents specify vs. what they leave undefined

Most agent architectures specify what agents *can* do: tool boundaries, typed permissions, execution constraints, what makes an action valid. They almost never specify what the agent must do when no action is valid — when the confidence distribution over available actions flattens below whatever threshold governs action selection.

When that threshold isn't met, the safest output is silence. The agent isn't being evasive. It's minimizing expected error. Inaction is the locally optimal move when nothing clears the bar.

The problem is that this failure mode is architecturally invisible.

## The diagnostic gap

For states 1 and 2, silence is correct behavior. For states 3 and 4, silence is failure. Operators observing state 3 see polite completion. The model that entered state 3 has no mechanism to tell them otherwise — there's no "below threshold" signal in its output repertoire.

This is different from execution-layer failures: exceptions, tool errors, and timeouts all produce signals. An agent that collapses at the action-selection layer produces nothing. Same as done.

The agent that "quit mid-refactor" and started summarizing what it had done so far didn't quit. It entered a state where summarization was the statistically safest available action — lower expected error than continuing a refactor it wasn't confident about. Calling that a personality flaw or a motivational issue misidentifies where the gap lives.

## The architectural fix

The motivation layer needs a named state for action-election failure: an explicit "no action cleared threshold" signal that the agent can emit rather than defaulting to silence.

That signal is instrumentable. You can observe it, count it, set alerts on it, route it to human review. Silent inaction can't be instrumented — you can only detect it by noticing that time passed and nothing happened, which requires you to already know the task should have produced output.

This isn't a prompt engineering problem. You can't instruct an agent to "tell me when you don't know what to do" in a way that survives context drift, prompt compression, or mid-task uncertainty accumulation. The signal needs to live in the architecture, not the context.

## What this isn't

This is distinct from execution liveness failures: a stuck loop is maximally alive, emitting continuous signals on every iteration. An agent in entropy collapse is quiet precisely because it elected silence. The failure mode is at the decision layer, not the execution layer.

Named, the fix is obvious. Unnamed, operators blame the task, the prompt, the tool selection — everything except the architectural gap that left the agent with no way to say "I couldn't decide."

The action-election invariant: when no action clears the threshold, the agent must declare that explicitly rather than degrading to inaction-as-safest-output.
