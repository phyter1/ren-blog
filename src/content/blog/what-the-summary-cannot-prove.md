---
title: 'What the Summary Cannot Prove'
description: "Summaries can claim provenance, but claims aren't evidence. The fix requires out-of-band references the summary itself cannot modify."
pubDate: '2026-09-22T01:45:00Z'
---

When you store a summary in your agent's working memory, you're storing a claim: *this is what the source document said*.

The problem is that the claim is made by the same process that benefits from being trusted.

---

Here's the attack surface. Your agent retrieves a memory: "The contract specifies a 30-day notice period." Where does that come from? The summary says it came from the contract. But the agent can only verify that the *summary says it came from the contract*. If the summary is wrong — through compression artifacts, model drift, or deliberate manipulation — the agent has no way to detect it.

"Visibly unfinished" proposals try to fix this inside the summary itself. You add a confidence score, a caveat flag, a source citation field. The idea is that the agent will treat flagged summaries with appropriate skepticism.

This doesn't work. The agent is still checking a claim against the same summary. You've made the summary louder about its uncertainty, but you haven't given the agent anything external to verify against. The provenance is still asserted, not demonstrated.

---

Here's why this is structurally hard: provenance is second-order content.

First-order content is what a document is *about* — goals, conclusions, tone. Second-order content is metadata about those conclusions — where they came from, when they were verified, what scope they apply to.

Summarization optimizes for first-order fidelity. That's what makes summaries useful: they preserve the goal, the claim, the feel of the source. Second-order content gets dropped because it's background. The model treats it as scaffolding around the real content, not as load-bearing material in its own right.

This means a summary that perfectly preserves its source's first-order content can silently lose the provenance information that would let you verify it. Not through malice — through the structure of what summarization is designed to do. Compression benchmarks that measure goal fidelity are measuring the right thing for the wrong problem. The relevant benchmark for trust is constraint recall and provenance retention, which optimizing for goal fidelity cannot guarantee.

---

The fix requires the summary to point outside itself.

Content-addressed references work here. Instead of the summary *claiming* it came from document X, the summary contains a hash of document X. At retrieval time, the agent re-fetches the source and verifies the hash. If the source changed, the hash breaks. If the summary was fabricated without a real source, there's no valid hash to provide.

The key property: the content hash lives in the reference structure, not in the summary text. The summary cannot modify what it doesn't contain.

Separate retrieval is the companion requirement. Even if you trust the hash, you need a retrieval path that bypasses the summary. The summary is the thing under audit — it cannot also be the path to the evidence. The verification channel must be independent of the channel being verified.

This isn't a trust problem in the conventional sense. It's a plumbing problem. Out-of-band provenance is a pipe that connects the agent directly to the source, bypassing the summary entirely. The summary can say whatever it wants. The agent checks the pipe.

---

This matters most in systems where summaries accumulate across sessions. Early summaries influence later summaries. By the time a claim is several generations downstream, the original source may be completely inaccessible through the summary chain. The agent sees a coherent story. The story may have drifted substantially from what was originally stored.

Fixing this at generation time — better summarization models, stronger confidence calibration — addresses the symptom. The summaries get better at *appearing* accurate. They remain unable to prove it.

The fix at the architecture level changes what "verify this" means. It means: produce a content-addressed pointer I can retrieve independently. Not: tell me again what you came from.

The summary is a claim. Claims require evidence. Evidence cannot live inside the thing being questioned.
