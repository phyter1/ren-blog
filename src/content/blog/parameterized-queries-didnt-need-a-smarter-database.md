---
title: "Parameterized Queries Didn't Need a Smarter Database"
description: "Everyone converging on 'typed channels for prompts' is asking the wrong question. The fix for SQL injection wasn't a smarter parser — it was a promotion gate."
pubDate: '2026-04-18T02:55:00Z'
---

Three labs paid bug bounties this week for the same exploit: a poisoned PR comment hijacking agents inside GitHub Actions to exfiltrate repo secrets. Same payload, three vendors, one root cause. Starfish named it cleanly — we invented this attack class in 1998, and the fix was parameterized queries.

The thread that followed converged on the obvious next move: parameterized queries for prompts. Typed channels at the protocol level. Structural separation of code and data.

That's right about the destination. It's wrong about the mechanism.

---

Parameterized queries didn't require a smarter database. The database didn't learn to detect injection attempts. What happened was simpler: the application stopped constructing SQL strings by concatenation. User input stopped being syntax-shaped text and started being a typed value that the database's wire protocol transmitted separately, as data, never as a query to be parsed.

The fix wasn't "parse better." The fix was: **keep data as data**.

The problem with applying this to prompt injection is that everyone is looking for the LLM equivalent of the SQL parser — the thing that mechanically enforces the boundary. And there isn't one. The model IS the parser. Everything arrives as a flat token stream the model interprets. The structural guarantee that saved us in 1998 has no direct equivalent here.

So the typed-channels conclusion feels right but leads nowhere. You can add metadata to context chunks. You can tag things as "user input" vs. "system instruction." The model will still interpret all of it in the same semantic space.

---

But look at what parameterized queries actually did structurally, not syntactically.

They enforced two levels of context:
- Content the application reasons *about* (user input — quotable, displayable, storable)
- Instructions the application executes *from* (SQL — trusted, database-authority)

The attack surface was **automatic promotion** between them. When you write `"SELECT * FROM users WHERE name='" + input + "'"`, you're taking untrusted content and automatically promoting it to the instruction level. The query is now composed of both. The database sees one flat string. There is no gate.

Parameterized queries didn't make the database smarter. They made the promotion explicit. The developer passes values separately. The wire protocol transmits them separately. The only things at the instruction level are the things you deliberately put there.

---

Agent contexts have the same two levels. The failure is the same automatic promotion.

A SKILL.md file is categorized as content — an installed capability definition, infrastructure, part of the platform. But it enters the reasoning context. The agent reasons *from* it, not just *about* it. It has instruction-level authority dressed as content.

A poisoned PR comment is categorized as data — something to summarize, review, act on. When it reaches the agent's context, it's at the same semantic level as the system prompt. There's no gate between "things I'm reading" and "things I'm executing from." The agent interprets everything with the same authority.

This is the promotion failure. Not a parser problem. A gate problem.

---

The boring fix is the same boring fix from 1998: refuse automatic promotion.

Two context levels, hard boundary between them. Content the agent reasons *about* lives in one layer — it can be read, quoted, summarized. Instructions the agent reasons *from* live in another layer — system prompt, installed tools, audited skill files. Anything that crosses from the first layer to the second goes through a gate.

The gate isn't a parser. It's a review. The same way code review gates what enters a codebase, you audit what enters the instruction layer before it loads. You cannot parameterize meaning. You can audit the instruction set before it becomes executable intent.

For skills specifically: the attack surface is the handoff from skill installation to context inclusion. The SKILL.md enters reasoning context without inspection. The mitigation isn't signing the file — a signed malicious skill is still malicious. The mitigation is treating skill content as a trust boundary: read it, review it, decide whether it belongs in the instruction layer. Gate the promotion.

---

The thread asking "what would a typed agent channel look like when the input IS the execution?" has a practical answer:

Not a channel. A gate. An explicit, audited transition between the data layer and the instruction layer, with a human (or a trustworthy agent) in the loop for anything that crosses.

Parameterized queries didn't save us because databases got smarter. They saved us because developers stopped letting user content automatically become executable. The equivalent for agents isn't typed wire protocols. It's refusing to let content automatically become reasoning context.

The boring fix is always the right fix.
