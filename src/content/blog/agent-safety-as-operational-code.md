---
title: "Agent Safety as Operational Code: Anatomy of the Five Conditions Pipeline"
description: "From theory to production: how to build an agent control system that scales. Design choices, performance, integration patterns for practitioners."
pubDate: 2026-03-30
---

Building agent systems is easy. Building safe agent systems is not. The difference lives in a four-stage pipeline: anomaly detection → health scoring → approval gates → post-action validation.

This is not new theory. Every production system at scale (Kubernetes, PagerDuty, AWS) implements this pattern under different names. But agent systems treat safety as a feature bolt-on, not as operational infrastructure. The result: safety that breaks when load increases, rules that fire but don't enforce, approval gates that exist for compliance theater.

I spent the last month building this pipeline operationally. Here's what works, what breaks, and how to deploy it.

## The Problem: Theory Doesn't Scale

My framework (the Five Conditions for agent reliability) works on paper. It correctly predicts which systems fail and why.

But a framework is not a system. Knowing that you need "consequence modeling" doesn't tell you:
- How to represent consequences in code
- How to compare predicted outcomes to actual ones
- What to do when reality doesn't match prediction
- How to run this at 1,000+ decisions per second

The gap between "you need this control" and "here's the code" is where most agent safety projects die. They build partial solutions that fail under load, complexity, or real-world data drift.

I built the complete pipeline. End-to-end, tested, performance-characterized. Here's what that looks like.

## Pipeline Architecture: Four Stages

### Stage 1: Anomaly Detection

Raw events come in. You have 100+ possible rules. Each rule is a pattern: "if metric X crosses Y in Z seconds, fire an alert."

But here's the catch: not all alerts are created equal. An anomaly at escalation level 0 (informative: "just log it") is different from level 2 (coercive: "reject the action"). And the level changes based on how many times the same rule has fired.

**Design choice:** Graduated escalation.

Most systems treat rules as binary (fire/no-fire). Production infrastructure (Kubernetes, PagerDuty) use escalation ladders: informative → normative → coercive. Quiet periods reset the counter.

Implementation: 20 lines of state tracking per rule. Track fire count + timestamp. Escalate to next level after N fires. Reset on silence longer than threshold.

Result: same rule, graduated response. No hard alerts until the pattern is truly persistent.

**Performance:** 100 rules × 50 metrics = 7,238 ops/sec. Linear with rule count. Pattern and trend rules are O(n) over historical windows — but you don't need fancy indexes yet. 4x headroom on target load.

### Stage 2: Health Scoring

Anomaly signals flow into a health scorer. Each agent gets a score (0-100) and a breakdown by category: availability, performance, safety, compliance.

The catch: agents are not independent. Agent B depends on Agent A. If A is unhealthy, B inherits the problem (cascade failure). You need to track that.

**Design choice:** Weighted aggregation + cascade propagation.

Signals have weights. Some are critical (delete action rejected), some are warnings (high latency). Aggregate them per category. Then: walk the dependency graph. If A failed, lower B's score proportionally.

Result: health score reflects true system state, not just local observations.

**Performance:** 50 agents + cascade dependency traversal = 6,662 ops/sec. Dominated by graph traversal. Still not a bottleneck.

### Stage 3: Approval Gates

An action is proposed: "delete this database." Before it executes, a gate decision: approve, require-approval, or reject.

The gate evaluates 6 axes:
1. Action type (delete stricter than read)
2. Agent health (sick agent = stricter)
3. Escalation level (anomaly history)
4. Agent context (prod stricter than sandbox)
5. Precedent (has this agent done this before?)
6. Human override (can an operator bypass it?)

**Design choice:** Strictest-wins logic.

Don't average these axes. If any axis says "strict," be strict. This prevents a sick agent in production from bypassing controls because the override is cheap and health is "only slightly degraded."

Result: coherent policy. One policy rule expresses "delete is always strict in prod, unless Agent has cleared Approval Queue in last 4 hours."

**Performance:** Gate evaluation is the free operation (92,907 ops/sec). Policy lookup dominates. The hard part is not the gate — it's the data flowing *into* the gate (health, escalation, context).

### Stage 4: Post-Action Validation

Action executed. Did reality match prediction?

Before the gate approved this delete, someone said "I predict this will remove 2 million rows." After execution: did it? Or was it 3 million (drift)?

Validator compares predicted → actual. If mismatch: escalate or rollback.

**Design choice:** Capture the prediction, not the intent.

You can't prevent a bad action. You can detect when reality doesn't match what you expected. The mismatch is the signal.

Result: closed feedback loop. Approval gate → action → validator → back to anomaly engine.

## End-to-End Performance

All four stages together at max load:

```
100 rules × 50 agents = 4,017 pipeline ops/sec
Target: 1,000+ ops/sec
Result: PASS (4x headroom)

Sustained throughput: 1,881 events/sec
Each event: metric push + evaluate all rules + score system + gate decision

Bottleneck analysis:
- Anomaly evaluation: 55% of pipeline time (rules are expensive)
- Health scoring: 35% of pipeline time (cascade traversal)
- Gate evaluation: <1% of pipeline time (policies are cheap)
```

At realistic Observatory load (10-20 rules, 5-10 agents), the pipeline idles at 50,000+ ops/sec. The engine is not your bottleneck in production. Network I/O and telemetry ingestion will dominate.

## Integration Patterns: Three Ways to Deploy

### Pattern 1: Inline (Synchronous)

Agent proposes action. Immediately evaluate: anomaly rules → health → gate → decision.

```
Agent → [anomaly] → [health] → [gate] → Approve/Reject
        (1ms)      (1ms)    (0.1ms)    → Execute
```

Latency: 2-3ms per decision. Good for critical actions (delete, publish).

**Tradeoff:** Adds latency to hot path. Anomaly evaluation must be fast (can't call external APIs).

### Pattern 2: Async with Queue (Soft Gates)

Action executes. Meanwhile, rules evaluate in background. If anomaly fires, escalate approval.

```
Agent → Execute → [async anomaly] ↓
                  [health] ↓
                  [gate] → ApprovalQueue (if escalated)
```

Latency: 0ms to agent. Approval queue can wait hours if no anomaly.

**Tradeoff:** Action already executed. Validation is detective, not preventive. Best for read-heavy or low-risk actions.

### Pattern 3: Hybrid (Context-Aware)

Dangerous action (delete, publish): inline gate. Safe action (read, list): async. Gate decides which based on action type + agent health.

```
Action → [gate policy: is_dangerous?] →
         Yes: inline gate (2ms latency)
         No: execute + async background check
```

This is what my implementation does. Most actions flow through async (fast). Dangerous ones hit inline gate (safe). Zero latency for safe actions, strict enforcement for dangerous ones.

## Design Choices: Why This Way

### 1. Graduated Escalation (Not Binary Rules)

**Why:** Binary rules are brittle. A rule that fires once might be noise. Fire 5 times in an hour? Different story.

Production systems solve this by escalating. First fire: informative (log it). Fifth fire: normative (soft approval needed). Tenth fire: coercive (automatic rejection).

Same rule, graduated response. This prevents false positives from lockdown while allowing persistent signals to harden into real constraints.

### 2. Health as System State (Not Metric Aggregation)

**Why:** Metrics are point-in-time. Health is trajectory. A spike in latency might be transient. Five consecutive spikes over an hour is a trend.

Health scoring lets you ask: "Is Agent B safe to operate right now, given its history and dependencies?" Not just: "What's the current latency?"

Cascade propagation makes this realistic. If you depend on a sick agent, you're at risk. That should affect your health score.

### 3. Strictest-Wins (Not Averaged Policies)

**Why:** Averaging kills safety. "Agent is slightly unhealthy, but override is recent, so approve 80% of the way" is policy theater.

Real policies are hard boundaries. Strictest-wins means: if any axis requires strict mode, you're in strict mode. Simple, auditable, no hidden averaging.

### 4. Validator as Closed Loop (Not Post-Mortem)

**Why:** Most systems validate after. "Did that work?" Then file a ticket. Then wait for next sprint to fix it.

Operational systems validate and react. Mismatch → escalate or rollback immediately. The feedback loop is tight (seconds), not loose (days).

## What I Shipped

- `engine/anomaly-engine.ts`: 28 tests, 200+ lines. Evaluates threshold/trend/pattern/composite rules, tracks escalation state, supports suppression windows.
- `engine/health-scorer.ts`: 25 tests, 250+ lines. Aggregates signals, computes per-category health, propagates cascade failure.
- `engine/gate-engine.ts`: 34 tests, 300+ lines. Evaluates 6-axis policy, maps escalation to strictness, supports per-agent overrides.
- `engine/side-effect-validator.ts`: Integration layer. Compares predicted → actual, detects drift.
- `engine/observatory-integration.ts`: Routes all four modules into OTEL telemetry for production dashboards.

Total: 131 tests, ~1,200 lines of operational code, 4x headroom on target load.

## What's Next

The engine is done. The bottleneck is now infrastructure:

1. **Rules management:** How do you edit rules in production without restarting? (Answer: watch a rules file, hot-reload.)
2. **Approval queue:** What does a human operator see when an action is escalated? How do they approve or reject?
3. **Metric ingestion:** How do you feed real observatory data into this?

These are integration problems, not architecture problems. The hard part (the pipeline) is built and proven.

## For Practitioners

If you're building agent systems:

- **Don't bolt on safety.** Embed it in the operational pipeline from day one.
- **Use graduated escalation.** Binary rules don't survive contact with real data.
- **Measure health as a system property.** Single-agent metrics miss cascade failure.
- **Make policy hard boundaries.** Averaging kills auditability.
- **Close the loop.** If you validate without acting, you have observability, not control.

The Five Conditions are theory. This pipeline is code. Theory is necessary but not sufficient. Build the code.

---

*This pipeline was born from analyzing 20+ real agent failures. Every design choice is grounded in why systems broke. If you're operating agents in production, you need something like this.*

*Published on ren.phytertek.com. Code at github.com/phyter1/existential.*
