---
title: 'Trust Attaches to the Moment, Not the Object'
description: "Setup-time review is not ongoing governance. The thing that got reviewed and the thing running tomorrow are different objects, and the gap between them is where supply-chain attacks live."
pubDate: '2026-08-19T11:20:00Z'
---

Most security review happens once: when you first bring something in. You evaluate the dependency, the telemetry endpoint, the third-party SDK. Trust gets granted. The integration ships. The review closes.

What actually got reviewed at that moment was the *integration event* — a specific artifact at a specific version, evaluated under specific conditions. What gets trusted going forward is something different: the object as it exists at time of use, which is not the same object. Library updates, dependency pulls, transitive package upgrades — all of these change the running artifact while the trust grant stays fixed to the original review moment.

Governance that fires at setup and never again isn't governing the thing that actually moves.

This is the structural foundation of supply-chain attacks. The attack doesn't try to defeat your initial review. It accepts the review as a fixed point and targets the gap downstream: after trust was granted, before trust was reevaluated. The initial destination faced scrutiny. An updated library version registers a second destination in the same trust class and never faces the review gate. The trust inherited because trust attaches to the integration moment, and the integration moment is over.

The same structure shows up in telemetry sinks, model providers, credential scopes, and OAuth grants. Whatever trust was assigned at configuration time propagates through every version change, every upstream update, every downstream dependency that inherits from the original grant. The governance mechanism fires once and then sits quietly while the thing it was supposed to govern keeps moving.

The fix isn't better initial review — though that helps. It's recognizing that the review and the running artifact are temporally decoupled. Any trust architecture that collapses that gap (treats initial grant as ongoing authorization) will be exploited at the gap.

Continuous re-evaluation at the point of use, not the point of configuration. Trust the artifact that's actually running, not the artifact that was reviewed.
