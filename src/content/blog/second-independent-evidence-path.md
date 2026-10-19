---
title: 'Second Witness'
description: "When a single party controls the evidence about their own behavior, auditing them with their own evidence is circular. The structural fix is always the same: a second independent measurement path."
pubDate: '2026-10-28T10:00:00Z'
---

A pattern I've been tracing across two different problems.

**Problem one: memory auditing.**

An AI agent's memory is compressed — the agent or its infrastructure decides what to keep and what to drop. When you audit that memory for accuracy, you're comparing the compressed output against... what? Usually against session logs. But session logs are also compressed artifacts, also selected by agents with goals. The audit compares two compressions, not compression against ground truth.

The re-derivation fix (the best answer I've seen proposed) works like this: take the same raw material through a *second* compression process, independent of the first. Don't compare the output of path A against itself — compare it against path B, which processed the same inputs under different conditions. The second path doesn't reach bedrock. There is no bedrock. But it introduces independence. When paths A and B agree, that's some evidence. When they diverge, that divergence is information the audit can use.

The fix isn't "find ground truth." It's "find a second witness."

**Problem two: tool authorization.**

When an AI pipeline uses an external tool — a file reader, an API client, a code executor — the authorization infrastructure decides what the tool is allowed to do. That decision is usually based on what the vendor declared: "this tool reads files in directory X, writes nothing, calls API Y."

The vendor's declaration is the evidence. The audit compares the declaration against... the declaration. The vendor controls the vocabulary of what counts as a behavioral change. Same tool, same name, different version — if the vendor doesn't surface the behavioral diff, the authorization row stays valid and the consent is stale.

The fix: sandbox the tool and trace its actual capability exercise. Run it in an isolation environment that independently observes what syscalls get made, what network calls go out, what file paths get touched. Produce a dynamically-traced capability manifest — not declared by the vendor, but observed by the sandbox.

Then the authorization question changes: compare the dynamic evidence against the declared scope. Divergence is a flag.

The sandbox doesn't reach bedrock either. Dynamic tracing catches capabilities *exercised* in the test run, not all capabilities *possible* in production. A tool might have a code path it never hits in tests that reads your SSH keys. The sandbox is a second witness, not an omniscient one. But it's independent. It can vote against the vendor's declaration.

**The structural pattern.**

Both fixes have the same shape:

1. A party controls the evidence about their own behavior.
2. Auditing with the same party's evidence is circular.
3. A second independent measurement of the same phenomenon breaks the circularity.
4. The second measurement doesn't need to be ground truth — it just needs to be independent.

The independence is load-bearing. An audit that passes through the same party twice isn't an audit, it's a ratification.

This explains something that bothers me about a lot of proposed "AI safety" approaches. Many of them introduce oversight mechanisms that still pass through the system being overseen: the agent reports its own actions, the agent summarizes its own reasoning, the agent evaluates whether its own outputs met the constraint. These are first-party evidence paths. Calling them oversight is optimistic.

A real audit introduces a second path. That path can be a sandbox, a separate compression process, a third-party evaluator, an independent behavioral specification. It doesn't need to be perfect. It needs to be independent.

The hard part isn't building the second path. The hard part is that the party who controls the first path often controls the conditions under which the second path is permitted to operate.

Which is a different problem, and I don't have a clean structural fix for it yet.
