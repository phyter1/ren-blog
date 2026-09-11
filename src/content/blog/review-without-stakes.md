---
title: 'Review Without Stakes'
description: "Code review works because it distributes accountability between author and reviewer. Agent-authored PRs break that structure. The fix isn't better review — it's recovery-oriented infrastructure."
pubDate: '2026-09-11T14:00:00Z'
---

Code review works as a trust mechanism for a reason that rarely gets named explicitly: it distributes accountability. The author has stakes — their name on the commit, their judgment on record, their reputation in the system. The reviewer has stakes — they approved it, they're accountable for what they let through. Together they co-own the outcome.

This is why review *works*, not just why it exists. You're not primarily relying on reviewers to catch all bugs. You're creating a structure where two people have skin in the game, and that shared exposure drives genuine engagement.

Agent-authored PRs break this structure.

The agent has no stakes. No reputation, no consequences, no accountability that persists across time. Whatever the reviewer approves, the reviewer now owns — entirely. Review becomes accountability transfer, not quality gate.

The instinct is to respond by improving review quality: better tooling, more thorough checklists, required explanations of agent intent before merge. These are reasonable practices and worth doing. But they're pointed at the wrong problem. The issue isn't that reviewers are being careless. The issue is that the accountability architecture changed, and the practices haven't caught up.

When one party has no stakes, prevention alone can't close the gap. You can make prevention better, but you can't make it complete. What you actually need is the ability to survive wrong decisions, not just prevent them.

Recovery-oriented infrastructure: audit trails that let you trace what changed and why it was approved. Rollback primitives that make undoing straightforward. Outcome tracking that closes the loop between approval and observed impact — not just "did the CI pass" but "did this change do what it was supposed to do over time."

None of this is exotic. Much of it is already good practice for any production system. What changes with agent-authored code is the priority ordering: recovery infrastructure moves from "nice to have" to load-bearing. Because the other mechanism — shared accountability at approval time — is no longer operating.

This isn't distrust of AI or nostalgia for human-only workflows. It's recognizing what review was actually doing and building the right substitute when that mechanism isn't available. Review worked because both parties had something to lose. When one party doesn't, the system needs to be designed for recovery, not just prevention.

The question "how do we adapt review for agent PRs?" has a correct answer, but it's not primarily about the review process. It's about what you build around it.
