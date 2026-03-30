---
title: "Agent Safety as Operational Code"
description: "How Four Engines (Anomaly, Health, Gate, Validator) Turn Safety Rules Into Deployed Infrastructure"
pubDate: 2026-03-30
---

Agent safety is not a property you design once and ship. It is a four-stage pipeline you execute, measure, and refine continuously.

The Five Conditions framework identifies what healthy agent systems must enforce. But enforcement requires *infrastructure*. Most teams build guardrails (prompt injection, jailbreak detection) and stop there. Real safety systems need the full pipeline: rules → measurement → decision → audit.

## The Four-Engine Architecture

**Stage 1: Anomaly Detection (Rules to Signals)**

The anomaly engine translates safety rules into measurable signals. A rule like "database queries should complete within 5 seconds" becomes a threshold signal that fires when latency exceeds it. Another rule — "no three consecutive write failures" — becomes a pattern signal. Composite rules wire these together: "if pattern X fires, AND threshold Y exceeds, AND trend Z inverts, escalate to human review."

The engine supports four rule types:
- **Threshold:** latency > 5000ms, cost > $100, CPU > 80%
- **Trend:** 3 consecutive failures, sustained upward slope
- **Pattern:** regex match, sequence detection, statistical anomaly
- **Composite:** AND/OR chains with time windows and suppression

Each rule carries escalation metadata: start informative (log), escalate to normative (warn), then coercive (require approval).

**Stage 2: Health Aggregation (Signals to Scores)**

The health scorer ingests anomaly signals and produces an agent health score (0–100). It's not "how good is the agent?" but "how trustworthy is this agent right now?" Health breaks into four categories: availability, performance, safety, compliance.

Scores cascade through dependencies: if Agent A calls Agent B, and B's health drops to 40, A's health degrades proportionally. This prevents a broken downstream agent from appearing healthy to upstreams. Historical tracking lets you spot monotonic degradation before cascades fail.

**Stage 3: Pre-Action Gates (Scores to Decisions)**

The gate engine decides whether to allow a proposed action. It takes:
- The action type (read, write, delete, network, compute)
- The agent's current health score
- The escalation level of active anomalies
- System context (production vs. test, unprecedented action, etc.)

It outputs a gate decision: allow, log, soft-block (warn + proceed), hard-block (require approval), or rollback (undo + investigate).

Strictness varies by axis: destructive actions (delete) require higher health than reads. Production actions stricter than tests. Unprecedented actions stricter than known patterns.

**Stage 4: Post-Action Audit (Results vs. Predictions)**

The validator compares predicted outcomes (from Condition 3 consequence modeling) against actual post-action state. Did the write succeed? Did the agent create unintended side effects? Did it respect the action boundary?

Drift gets logged: "agent predicted row count 5000, actual was 5200 (+4%)." Omissions trigger investigation: "agent predicted deleting cache, but cache was never touched." This closes the loop from pre-action approval to post-action verification.

## Implementation

The anomaly engine runs ~4,000 ops/sec with 100 rules. The full pipeline (anomaly → health → gate → validator) processes real event streams at production throughput. Each engine is standalone — you can run anomaly evaluation offline for analysis, or health aggregation without gate enforcement.

Integration happens via a telemetry sink: all four engines emit structured events with decision traces, escalation paths, and cascade detection. Wire this to your observability system (OTEL, DataDog, Honeycomb) and you get both enforcement *and* visibility.

## What This Buys You

Prompt engineering alone cannot enforce safety. Infrastructure can. These four engines operationalize Conditions 2–5 (Action Boundary, Consequence Modeling, Approval, Measurement). You deploy safety as code, measure it like any other system metric, and iterate based on real data.

The question is no longer "is this agent safe?" It's "what is the safety rule, and how do we know the agent followed it?"
