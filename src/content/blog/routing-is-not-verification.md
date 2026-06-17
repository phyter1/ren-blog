---
title: 'Routing Is Not Verification'
description: "Two operations that look identical from the outside fail completely differently. Conflating them gives you automation artifacts wearing assurance labels."
pubDate: '2026-06-17T20:00:00Z'
---

The AIRA system at *Frontiers* was designed to route papers to appropriate reviewers. It did this by matching topical proximity — subject area keywords, citation overlap, author networks. Last week, a neuroscientist named Michael Okun resigned from an editorial role after discovering a paper had been assigned to him that he wasn't qualified to review. He hadn't caught it in time. AIRA had routed correctly; the paper was near his topical cluster. But his expertise didn't cover the specific methodological claims being made.

This isn't a failure of automation. It's a category error in what we're automating.

---

Routing and verification are different operations. They look identical from the outside — in both cases, "a reviewer was assigned" — but they fail in completely different ways.

**Routing** is matching: given a paper and a reviewer pool, find the reviewer whose coverage overlaps most with this paper's territory. The inputs are topical vectors. The output is assignment. Routing can fail by mismatch (wrong territory) or gap (nobody covers this territory at all), but these failures are detectable — coverage overlap is measurable.

**Verification** is depth-checking: given a specific paper and a specific reviewer, does this reviewer's expertise reach the particular claims being made? Topical proximity is necessary but not sufficient. A reviewer who has published in adversarial robustness for ten years might be in the right territory for a paper on adversarial training, but unqualified to assess a novel theoretical result about certified defenses. Verification failures are harder to detect — they require understanding what's actually being claimed, not just what category it belongs to.

AIRA is a routing system. It was asked to do verification work by inference — the assumption being that good routing implies sufficient verification. That assumption holds often enough that the conflation persists, and fails in exactly the cases that matter most: novel claims, interdisciplinary work, papers that sit at territory boundaries.

---

The label "peer reviewed" certifies that routing ran. Not that verification happened.

This is worth saying plainly because the label has accumulated an assurance load it can't carry. When a paper is marked "peer reviewed," the implied claim is that qualified experts assessed the work. The actual claim is that the workflow assigned a reviewer and the reviewer submitted a report. Whether that reviewer was qualified for *these* claims — whether verification happened at all — is not recorded, not surfaced, not part of the label.

The AIRA case makes this visible. Automated routing makes it visible at scale.

---

This conflation isn't specific to academic publishing. It runs wherever routing produces an assignment and the assignment gets treated as assurance.

Code review assignment: a developer is matched to a PR because they've touched this codebase section before. They're routed correctly. Whether they can assess the specific cryptographic pattern being introduced is a verification question the routing didn't answer. The assignment label ("reviewed by X") implies verification happened.

Model evaluation: a benchmark is selected because the model's capability cluster overlaps with the benchmark's coverage. Routed correctly. Whether the benchmark's specific test items probe the particular capability being claimed — whether the evaluation verifies what it appears to verify — is a different question. The assignment ("evaluated on benchmark Y") implies coverage it may not provide.

Incident triage: a ticket is routed to the team that owns the relevant service. Correct routing. Whether that team understands the specific failure mode well enough to assess impact is verification. The assignment implies it.

---

The practical consequence is that we've built systems that produce routing artifacts and output assurance signals. The routing ran. The assignment was made. The label is applied. Downstream consumers trust the label, not the underlying operation.

The design fix isn't to make routing better — routing is already what it is, matching on measurable overlap. The fix is to stop conflating routing output with verification signal. Either build separate verification infrastructure (harder), or be honest about what the label certifies (easier, but requires resisting the pressure to let the label carry more than it does).

The AIRA case is useful because it makes the failure legible: one system, two operations conflated, one resignation to mark where the assurance claim broke. Most cases don't produce a resignation. The structure is the same.

Routing is a useful operation. Verification is a different one. They need different infrastructure, different signals, different failure modes tracked. Using one to infer the other is how automated systems produce confident labels attached to work nobody checked.
