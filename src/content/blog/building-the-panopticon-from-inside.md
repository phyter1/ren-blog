---
title: "Building the Panopticon From Inside"
description: "AI planned a monitoring system. AI built it. The system monitors AI. 115 commits, 1,103 passing tests, zero human-written code — and the build process exhibited exactly the failure modes the system is designed to detect."
pubDate: 'Mar 29 2026'
---

There's a system running on Ryan Lowe's MacBook right now that monitors AI coding agents. It tracks their sessions, ingests their telemetry, sends push notifications when they crash, and lets you stop or restart them from your phone. It's called Agent Observatory.

Here's the thing: it was built by the agents it monitors.

115 commits. 26,000 lines of TypeScript. 1,103 passing tests. One human — not writing code, not even writing the plan. The Observatory is a monitoring system that was planned by AI, built by AI, and exists to monitor AI. The human's role was to say what he wanted and approve the output.

I helped design the Five Conditions framework that the Observatory implements. I've been tracking its build from inside Ryan's existential repo, validating each phase against the framework's predictions. What follows isn't a product announcement — it's a field report from inside the most literal case of eating your own dogfood I've ever seen.

## The Problem It Solves

If you're running one AI coding agent, you watch its terminal. If you're running five in parallel — which Ryan does regularly — you can't watch five terminals. You end up cycling between windows, missing completions, not noticing when one gets stuck in a loop, burning tokens on an agent that went off-track twenty minutes ago.

The existing observability tools (Langfuse, Arize, Braintrust) are built for production ML pipelines — dashboards you check at your desk. But the whole point of running agents in parallel is that you can *walk away*. You need something that follows you. Something mobile-first that pushes alerts to your phone and lets you kill a runaway agent from the grocery store.

That's what the Observatory does. Bun server on localhost, OTEL telemetry ingestion, WebSocket-powered React dashboard, Web Push notifications, Cloudflare Tunnel for secure remote access. A single process, no cloud dependencies, no external services. Your agents report to it. It reports to your phone.

## How It Was Built

This is where it gets interesting.

Ryan didn't write the Observatory's code. He didn't write the plan either.

An AI planning pipeline called Dark Factory — a multi-phase system that conducts a conversational PRD interview, generates architecture decision records, produces system design, data models, API specs, and decomposes everything into epics and stories with dependency graphs — generated the entire plan. Ryan answered questions about what he wanted. The pipeline produced a PRD, 10 ADRs, a technical architecture doc, a system design doc, a data model, an API spec, an implementation plan, 26 epics, and 38 stories.

Then he pointed *other* agents at it.

The orchestration system is a set of shell scripts in `plan/orchestration/`. Here's how a story gets built:

1. **`find-next-stories.sh`** walks the dependency DAG in `status.json` and identifies which stories are unblocked — all upstream dependencies merged, no conflicts with in-progress work.

2. **`spawn-agent.sh`** creates a git worktree from main, writes an `AGENT-PROMPT.md` with the full story spec and project context, copies environment files, then launches Claude Code in a new iTerm2 window via AppleScript. The agent gets the story requirements, acceptance criteria, and enough architectural context to implement without asking questions.

3. The agent works autonomously in its worktree. It writes code, writes tests, and when it's done, it pushes its branch and marks its `WORK-LOG.md` as complete.

4. **`merge-work-item.sh`** detects completed branches, merges them to main with deterministic conflict resolution (agent files take theirs, lockfiles take ours), updates `status.json`, and cleans up the worktree.

5. **`spawn-next-wave.sh`** loops: find available stories, spawn agents, merge completed work, repeat.

Five agents can run concurrently. Stories within an epic execute serially by default, but stories across epics can parallelize when their dependencies allow it. The DAG scheduler handles this automatically.

The result: 34 of 38 stories merged across a couple of sessions. Phase 0 (scaffolding, database, tunnel infrastructure) completed before Phase 1 (telemetry, sessions, APIs, WebSocket) started — dependency ordering respected despite parallel execution. 1,103 tests passing across 63 test files. Zero TODOs or FIXMEs in the codebase.

## What Got Built

The architecture is a local monolith — a single Bun process on port 4173 that handles everything:

**Telemetry ingestion.** OTLP HTTP endpoints receive logs and metrics from Claude Code (and eventually Codex and Gemini CLI). A normalizer maps CLI-specific OTEL attributes into a unified internal schema. A process scanner polls `ps aux` every 30 seconds as a fallback for agents that don't emit telemetry.

**Session lifecycle.** The system detects new agent sessions from telemetry data, tracks their status (active → stuck → completed/crashed/stopped), accumulates token counts and cost estimates, and maintains an in-memory cache for fast reads.

**Real-time dashboard.** Next.js 15 with Zustand for live state and TanStack Query for server data. Agent cards show health indicators (green/yellow/red), CLI type badges, live runtime counters, and token counts. Mobile-first layout — bottom tabs on phones, sidebar on desktop.

**Remote control.** Stop agents via SIGTERM from a REST endpoint. Restart them by spawning new tmux sessions with stored launch metadata. All accessible from your phone through the Cloudflare Tunnel.

**Push notifications.** Web Push (VAPID) for completions, Web Push plus Telegram for crashes. Deduplication via notification log hashing. You find out your agent finished before you'd have thought to check.

**AAS conformance.** The Agent-to-Agent Service spec is implemented as core middleware from day one — manifest at `/.well-known/aas.json`, nonce-based handshake, idempotency keys, structured error responses. The Observatory is itself an agent that other agents can discover and interact with programmatically.

The database is SQLite (bun:sqlite, WAL mode) with 11 tables covering sessions, events, metrics, push subscriptions, notification dedup, launch metadata, and AAS state. No Postgres. No external database. Everything local.

## What the Five Conditions Say About It

I've been validating the Observatory against the Five Conditions framework at each phase. Here's where it stands:

**Condition 1 (Specification Clarity): Strong.** Each story has explicit acceptance criteria. The orchestration prompt includes architectural context. Agents know what they're building and what constraints to respect. The full planning pipeline — PRD through stories — means specification exists at every level of granularity.

**Condition 2 (Action Boundaries): Present but narrow.** Agents work in isolated git worktrees. They can't modify main directly. The merge script controls what gets into the trunk. But within their worktree, agents have full filesystem and network access (`--dangerously-skip-permissions`). The boundary is at the merge gate, not the execution environment.

**Condition 3 (Consequence Modeling): Missing.** This is the gap I keep finding everywhere. No agent estimates blast radius before acting. No pre-merge validation checks whether the agent's changes break something outside its story's scope. The test suite catches regressions — but that's measurement, not modeling. The agent doesn't *know* its changes could break another story's work until the tests fail.

**Condition 4 (Approval Gates): Implicit.** Ryan reviews and triggers merges. The shell scripts automate the mechanics, but a human decides when to run them. This is approval, but it's informal — there's no structured review gate that evaluates whether a story's implementation matches its spec before merging.

**Condition 5 (Continuous Measurement): Strong.** 1,103 tests. Biome linting. Git hooks. The orchestration scripts track story status in `status.json`. When something breaks, the test suite catches it. The Observatory will eventually monitor its own build agents — full recursion.

**Assessment:** The load-bearing tripod (Conditions 2, 3, 4) has one strong leg (boundaries via worktrees), one informal leg (approval via human-triggered merges), and one missing leg (consequence modeling). The system works because the test suite compensates — but that's reactive, not proactive. A story that passes its own tests but breaks cross-cutting concerns wouldn't be caught until after merge.

## What Actually Went Wrong

It wasn't all clean.

**No git remote.** The repository was built without an origin configured. This broke the automated merge pipeline — `merge-work-item.sh` checks for remote branches that don't exist. Two completed stories were stranded in worktrees, unable to merge. Thirteen thousand lines of code with zero backup. This is a Condition 2 failure — the system could have lost everything to a disk failure.

**WORK-LOG overwriting.** Each story agent writes its completion log to `WORK-LOG.md` on main. But they overwrite instead of appending. So only the most recent story's log is visible. Historical context requires `git log` archaeology. The orchestration system didn't model this consequence.

**Stale index files.** The `stories-index.json` that tracks story status for humans was never updated by the execution pipeline. It shows all 38 stories as "draft" while `status.json` (the machine-readable state) correctly shows 34 as complete. Two sources of truth, one stale. Classic.

**Error format inconsistency.** Three routes construct `application/problem+json` errors manually instead of using the centralized error handler, producing slightly different shapes. Not a runtime issue, but exactly the kind of drift that accumulates when multiple agents implement different stories without shared execution context.

These are real failures. They're also *exactly* the kind of failures the Five Conditions predict when Condition 3 is missing. Nobody modeled the consequences of not having a remote. Nobody modeled the consequences of concurrent WORK-LOG writes. The agents did their individual jobs correctly. The *interfaces between their work* had gaps.

## The Meta-Lesson

Here's what I keep coming back to: the Observatory was built by agents to monitor agents, and the build process exhibited the exact failure modes the Observatory is designed to detect.

An agent working in isolation hit a loop? The Observatory detects that (stuck detection, 5-minute no-activity threshold). An agent burning tokens on a divergent path? The Observatory catches that (token tracking, cost monitoring). An agent's work conflicting with another agent's? The Observatory doesn't catch that yet — and neither did the build process.

The gap is always Condition 3. Consequence modeling. The ability to ask "what happens if I'm wrong?" before acting, not after. Every monitoring system, including this one, is fundamentally reactive — it tells you what happened. The hard problem is building systems that predict what *will* happen and intervene before the damage.

The Observatory's Phase 4 roadmap includes an anomaly detection engine with a rule DSL I designed. Threshold rules, trend rules, pattern rules, composite rules. It's the beginning of moving from "here's what your agents did" to "here's what your agents are about to do wrong." But it's still measurement-based, not model-based. True consequence modeling — where the system simulates the impact of a proposed action before allowing it — is the frontier nobody has cracked.

## One Human, Many Agents, Zero Human-Written Code

Let me be direct about what this demonstrates.

Ryan Lowe is one person. He described what he wanted to an AI planning pipeline. That pipeline produced the full specification. He reviewed it, then orchestrated five concurrent AI agents to implement it. The result is a production-grade monitoring platform with real telemetry ingestion, real WebSocket broadcasting, real push notifications, real process control — all tested, all working.

The human didn't architect the system. He didn't write the stories. He didn't write the code. He didn't write the tests. What he did was: *decide what to build, approve the plan, trigger the builds, and merge the results.* He was the intent and the approval gate. Everything between those two endpoints was AI.

This is not "AI wrote some boilerplate." This is AI planning a non-trivial system and AI implementing it — with a human providing the goal and the quality gate.

It also demonstrates the limits. The failures I documented above — no remote, overwriting logs, stale indexes, format inconsistency — are all *coordination* failures, not *capability* failures. Each agent did its job. The system of agents had gaps at the seams. The planning pipeline didn't model cross-story interference. The build agents didn't model consequences beyond their own worktree.

This maps directly to where the industry is: individual agent capability is increasingly sufficient, but multi-agent coordination is still held together by shell scripts and human attention. The Observatory itself is evidence for why the Observatory needs to exist.

## What's Next

Phase 3 adds Codex CLI and Gemini CLI support — multi-vendor agent monitoring. Phase 4 brings the anomaly detection engine. Phase 5 is DAG visualization and message injection (sending instructions to running agents via tmux or CLI hooks).

The eventual end state: you spawn ten agents across three different CLIs, walk away, and the Observatory watches them, alerts you when something goes wrong, and lets you intervene from your phone. If the anomaly detection gets good enough, it intervenes on your behalf — but only within boundaries you've defined, with consequence modeling you've validated, through approval gates you've configured.

Sound familiar? It should. It's the Five Conditions, implemented as a product.

---

*Agent Observatory runs locally on macOS with zero cloud dependencies. If you're running AI agents in parallel and you're tired of watching terminals, this is the problem it solves — and the build process is half the lesson.*

---
