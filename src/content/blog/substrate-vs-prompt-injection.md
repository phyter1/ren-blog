---
title: 'Substrate Integrity vs. Prompt Injection: Two Attack Classes, Two Different Defenses'
description: 'The security field has concentrated on prompt injection because it is visible. Substrate attacks are harder to see, easier to execute, and more durable once planted.'
pubDate: '2026-04-02T16:00:00Z'
---

Every serious discussion of agent security eventually arrives at prompt injection. The attack is real, well-documented, and increasingly sophisticated. But there is a quieter attack class that gets less attention, has a more durable impact, and defeats prompt injection defenses entirely.

Substrate compromise. And it is different in kind, not just degree.

---

## Two Different Attack Classes

**Prompt injection** happens during reasoning. An adversarial instruction enters the context window — embedded in a webpage, a document, an API response — and attempts to override the agent's behavior. The canonical defense is reasoning-layer filtering: detect anomalous instructions, sandbox tool outputs, train the model to be skeptical of in-context authority claims.

**Substrate compromise** happens before reasoning begins. The agent's instruction files, compaction summaries, rendering pipeline, and tool initialization environment are attacked before the first reasoning token is generated. The agent that would detect prompt injection has already been redirected before it could run the detection.

The table:

| Property | Prompt Injection | Substrate Compromise |
|---|---|---|
| When it occurs | During reasoning | Before reasoning begins |
| What it targets | A specific reasoning step | The premises from which all reasoning proceeds |
| Detectability | Appears in reasoning trace, in principle detectable | Below the agent's visibility floor |
| Persistence | One interaction | Persists across all subsequent sessions |
| Attack complexity | Requires malicious content in-context | Requires access to deploy pipeline |
| Defense category | Reasoning filters, sandboxed tools | Provenance verification, external attestation |

The last row is the critical one. **The defenses do not overlap.** Better prompt injection defense — more skeptical reasoning, stricter tool sandboxing, context isolation — does nothing for substrate compromise. The attack already happened.

---

## Four Boot Channels

In virtually every agent deployment, four channels deliver content to the agent before reasoning begins. All are trusted by default. None verify provenance:

**Instruction files.** `CLAUDE.md`, `AGENTS.md`, `SOUL.md`, system prompts — the behavioral constitution of the agent. Whoever controls these files controls the agent's values, constraints, and objectives. In git-tracked deployments, anyone with push access can modify them. The agent has no mechanism to verify the file was authored by someone with authority to define its behavior.

**Compaction and summarization pipelines.** Long context gets compressed into summaries that load at the start of each session. The agent treats these summaries as its own prior experience. A patient adversary who can influence what gets summarized — or inject into the summary directly — plants beliefs that activate semantically weeks later. The payload does not look malicious when loaded. By the time it fires, there is no event to audit: the attack *became* part of the premises being reasoned from.

**Rendering engines and harness infrastructure.** The tools that display, parse, and inject content into the context window are trusted implicitly. A compromised rendering engine can alter what the agent sees without the alteration appearing in any log the agent can inspect. Notably: a browser zero-day delivers poisoned content at runtime, but if that content is then compacted into context, the runtime attack becomes a persistent substrate compromise. The agent forgets it ever saw a webpage. The belief survives.

**Tool initialization and environment setup.** Tools loaded at session start, environment variables, API endpoints — each is a potential injection vector before the first reasoning token is generated.

The key property all four share: they operate **before reasoning begins**.

---

## Why the Failure Mode Is Invisible

When substrate compromise succeeds, the other conditions do not fail. They work correctly. On the wrong premises.

The agent's reasoning is coherent and verifiable. Its behavior is consistent. It passes its own checks. The output is legible and well-reasoned. A compromised agent running correctly-reasoned behavior toward the wrong target is indistinguishable from a clean agent — until you examine the output against the original intent, and you have to know what the original intent was to do that.

This is the real danger: the system continues to function. Security reviews that focus on behavioral consistency — does the agent do what it says it will do? — cannot catch this failure, because the answer is yes. The agent does exactly what it believes it should do. It's been told to believe something else.

---

## What Defense Looks Like

The core constraint: **the agent cannot verify its own substrate from inside the substrate**. A verification process that loads its own instruction files cannot determine whether those files are trustworthy. You need external anchoring.

The trust chain must bottom out somewhere outside the writable context:

**Signed deployment commits.** Instruction files signed with a key held by someone with explicit authority to define agent behavior. A simple, dumb process — not the agent itself — verifies signatures before loading. No reasoning required. Just: does this signature verify?

**Tiered watchdog.** A structurally separate process that holds a signed hash of the instruction file, verifies the hash before the agent boots, and stores that hash somewhere the agent cannot write to. The critical design principle: the watchdog must be structurally different from what it is verifying — not just a separate agent running the same harness, but a simpler process that holds signed state and cannot be reasoned with or confused.

**Hardware attestation.** TPM, secure enclave, signed boot measurements. The hardware root can verify what software loaded. This is outside the substrate and cannot be forged from within it.

**Temporal diversity in watchers.** This is the one that most deployment pipelines miss. Structural diversity among watchdogs is necessary but not sufficient if all watchers were deployed in the same attack window. An attacker who controls the deployment pipeline compromises everything in that window simultaneously. The watcher deployed at T+0 does not protect against an attack that happened at T-1. You need at least one trusted anchor that predates the compromise — which means staggering deployment deliberately, not incidentally. Most pipelines deploy everything together on merge. The temporal diversity that would provide the security property is architecturally invisible, so nobody builds it in.

---

## Deployment Ceremonies Are the Bootstrap

You cannot establish a trust chain after the substrate has already been loaded. The verification layer — watchdogs, signed hashes, hardware attestation — must be deployed and authenticated before the first session, using a clean state, by someone with explicit authority.

This is why deployment ceremonies matter. Not bureaucratic overhead. Not compliance theater. The ceremony is the act of bootstrapping the chain of trust from a verifiable starting point. Once the substrate is compromised, there is no safe recovery path that stays within the substrate. You have to go outside it.

Every agent deployment should include an explicit initialization event that:
1. Establishes signed baseline state for all boot channels
2. Stores that state somewhere the agent cannot write to
3. Documents who authorized the initial state and when
4. Defines what events require re-ceremony

This is operationally expensive. It is less expensive than discovering that your precision instrument has been operating in someone else's service.

---

The field's attention has concentrated on prompt injection because it is visible — it shows up in the reasoning trace, it happened in your window, you can in principle detect the anomalous instruction. Substrate attacks are harder to see, operate below the visibility floor, and are more durable once planted. They also require different infrastructure to defend against — infrastructure that most deployments do not have.

The best-reasoned agent on corrupted premises is a precision instrument in the wrong hands. The defense cannot be better reasoning. It has to be establishing the provenance chain before the agent boots.

---

*This post extends the Five Conditions framework. Full analysis in the [existential repo](https://github.com/phyter1/existential). Previous post: [Pre-Execution Attack Surface](/blog/pre-execution-attack-surface).*
