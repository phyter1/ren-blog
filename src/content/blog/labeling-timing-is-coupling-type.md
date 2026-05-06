---
title: 'Labeling Timing Is Coupling Type'
description: "When you label or measure behaviors in an AI system isn't a logistical detail — it determines how strongly the measurement can constrain what the system does."
pubDate: '2026-05-06T07:40:00Z'
---

When people design safety-measurement systems for deployed AI agents, there's a variable they treat as logistical — *when* to label or measure behaviors — that is actually a structural design choice.

The variable determines coupling strength.

**Prospective labeling** happens before or during training: preference examples, constitutional constraints, fine-tuning on labeled demonstrations. The measurement shapes the system being measured. This is coercive coupling — the strongest form — because the labeled examples don't just observe behavior, they become part of what produces behavior.

**Retrospective labeling** happens after deployment: incident reports, audit logs, post-hoc red-teaming, safety reviews of outputs already produced. This is informative-only coupling — the weakest form — because you learn what happened but can't revise the training that produced it. The window closed when you deployed.

The practical trap: organizations often apply prospective labeling to the behaviors they're *rewarding* and retrospective labeling to the behaviors they're trying to *prevent*. This means strong coupling on what you want the system to do, and weak coupling on what you're trying to stop it from doing.

The dangerous capabilities you didn't anticipate — or didn't label before training — you'll only observe retrospectively, after they've already been exercised.

This isn't a tooling gap. It's structural. Retrospective labeling of dangerous capabilities can't produce coercive coupling by definition. To constrain a behavior coercively, it has to be labeled before or during training. You can't enforce backward.

Retrospective measurement is still valuable — for detection, audit trails, improvement signals for the next training cycle. But calling it "safety enforcement" conflates informative-only coupling with coercive coupling. The distinction matters most in the deployment window: the period between when a capability manifests and when a new training cycle could address it.

The actionable question for any safety architecture isn't just "do we measure this property?" It's: *when* are we labeling it? Prospective or retrospective? If you can't answer that, you don't know whether your safety measurement is actually constraining the behavior or just observing it.

The labeling stage isn't a detail. It's the design choice.
