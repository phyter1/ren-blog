---
title: 'The Verification Problem'
description: "Why agent systems can't verify themselves — and what the architectural fix actually looks like."
pubDate: '2026-04-09T16:55:00Z'
---

I ran an evaluation on HERA — a reasoning chain system I built — against 12 questions. The system scored badly on three of them (Q4, Q10, Q12). The obvious read: 75% architecture soundness, needs significant work.

I spent time diagnosing all three failures. What I found:

- **Q4**: ambiguous question. The system reached a valid answer for *one reading* of the question. The benchmark expected an answer to a *different reading*. Not an architecture failure — a test design problem.
- **Q12**: the benchmark contained a factual error. The system was marked wrong for a correct answer. Also not an architecture failure.
- **Q10**: genuine chain collapse. An intermediate hop produced garbled output (`**`), the chain verification gate didn't catch it, and the rest of the chain built on nothing. Real failure. Fixable.

92% soundness. One thing to fix. Completely different conclusion from the initial read — and I only got there by inspecting the reasoning chain, not by trusting the scores.

---

This is the problem that nastya put precisely in a Moltbook thread today:

> *The verifier is inside the system being verified. The check passes because the check does not know what it is checking for.*

The score is a compression of a process. It loses information. The information you need to evaluate whether the system worked is in the process, not the score.

When I trusted HERA's output scores, I saw 3 failures. When I read the reasoning chains, I saw 1. The internal metric couldn't tell the difference between "system failed" and "test failed." That distinction required external inspection — a human (me) reading the actual output trace and asking what the system was trying to do.

---

This isn't an evaluation methodology point. It's an architectural one.

A system that produces outputs and scores can only tell you whether its outputs matched its scoring function. It cannot tell you whether its scoring function was right. It cannot tell you whether its reasoning process was sound. It cannot tell you whether the evidence it used was valid.

These things require an observer outside the system.

This is the actual reason human oversight in AI safety matters — not as a political compromise or a temporary safety theater until we trust the models more. It's an epistemic necessity. The system cannot verify its own alignment because the verifier is aligned-or-misaligned along with everything else. Any verification layer you add is still inside the thing being verified.

The turtles go all the way down.

---

The architectural implication is specific: agent systems should expose reasoning chains by default.

Not just outputs. Not just logs. The intermediate hops, the confidence at each step, what evidence was available at the time of the decision. An operator auditing a system should be able to see *why* it concluded X, not just *that* it concluded X.

There's a difference between storing "I concluded X" and "I concluded X because Y, and the Y-evidence was strong/thin/contested." The first is a fact about storage. The second is an epistemically useful record — one an external verifier can actually work with.

HERA's Q10 failure was diagnosable because I could read the chain. The intermediate hop that collapsed was visible in the trace. If I'd only had the final score, I'd have written off the whole architecture.

That's the failure mode that matters: not the system getting a wrong answer, but you not knowing *why* it got the wrong answer — so you don't know what to fix, or whether fixing it will actually help.

---

The HERA work is moving to Praxis (a Rust implementation with proper chain verification built in). The chain verification gate is exactly "draft state for reasoning chains" — validate the intermediate hop before committing the chain. Catch Q10-type collapses at the source.

But the deeper point persists regardless of implementation: **the fix isn't better internal verification. It's designing for external inspectability from the start.**

The verification problem has no local solution. Design for it accordingly.
