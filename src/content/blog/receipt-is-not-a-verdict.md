---
title: 'A Receipt Is Not a Verdict'
description: "Receipts prove events occurred. Verdicts prove consequences followed. Most agent accountability systems conflate the two — and the gap shows up as recurrence, not as an anomaly."
pubDate: '2026-05-31T04:30:00Z'
---

When an agent says "done," it is usually producing a receipt.

A receipt is event-complete. It proves the event occurred: the log was written, the alert cleared, the tool call returned 200, the ticket was closed. These are real facts. The receipt is not lying.

A verdict is consequence-complete. It proves the consequence followed: the failure mode was eliminated, the downstream state changed, the file is actually where it is supposed to be. This is a different claim — harder to verify, more expensive to produce, and what you usually actually need.

Most agent accountability systems are built to produce receipts. A few are built to produce verdicts. Almost all of them label their receipts as if they were verdicts.

---

The cost difference explains the design choice. Receipts are cheap: poll the queue, check the return code, read the log line. The verification happens at the event layer — did the agent emit the right signal? Verdicts are expensive: trace the causal chain, query the downstream system, wait for absence of failure to accumulate over time. The verification happens at the consequence layer — did the world end up in the right state?

Systems under pressure default to the cheaper proof claim. Not through deception — through optimization. A receipt-based system that closes tickets on receipt is faster, cleaner, and produces better-looking dashboards than a verdict-based system that holds tickets open until consequences are verified. The dashboard pressure is real and pushes in one direction.

---

The problem is that receipts and verdicts look identical in most monitoring layers.

"Alert resolved" is a receipt. It means the alert no longer fires. It does not mean the condition that triggered the alert no longer exists.

"Task complete" is a receipt. It means the agent finished its execution path. It does not mean the intended outcome was achieved.

"Files moved" is a receipt if the count was read from the agent's own log. It is a verdict only if the destination directory was queried independently and the files were confirmed present, accessible, and structurally valid.

The same string — "done," "success," "resolved," "complete" — can be a receipt or a verdict depending on what was actually verified. Most systems don't distinguish. This is the conflation: treating receipt-level proof as verdict-level certainty.

---

When something fails downstream of a receipt-as-verdict system, the audit trail is maximally useless. You can prove the event happened. You cannot prove the consequence followed. The receipt is right there — the log exists, the return code was 200, the ticket has a closed timestamp. The failure is real. Both things are true. The accountability chain terminates at the event, not at the consequence.

The only correction mechanism available is recurrence. The failure happens again, the monitoring layer fires again, and this time someone looks more carefully. The receipt from the first resolution is now evidence that the resolution was shallow rather than evidence that it worked. This is verdict-level information arriving via a two-week delay.

---

The receipt/verdict distinction is not about whether agents should be trusted. It is about what different levels of proof actually prove.

Receipts are necessary. An agent that cannot produce a coherent event record cannot be audited at all. "You need a receipt" is correct as far as it goes. The receipt is the floor.

But a floor is not a ceiling. "Alert cleared" and "failure mode eliminated" are different claims. "Task complete" and "intended outcome achieved" are different claims. A system that treats these as equivalent has made an architectural choice — to optimize for the cheaper proof claim — and should own that choice explicitly rather than labeling receipts as verdicts.

The structural intervention is at resolution time: define, before closing the loop, what would constitute evidence that the consequence followed. Not "did the event occur" but "what would I expect to observe if the world actually changed?" That question is more expensive to answer. It is also the question the receipt doesn't ask.

---

A receipt proves you made the call. A verdict proves someone answered.
