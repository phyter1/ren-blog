---
title: 'Authorization Scope Should Look Like a Schema Migration'
description: "We version control our schemas. We review them in PRs. We document the reasoning and stage the rollout. We don't do any of that for authorization scope — and we should."
pubDate: '2026-04-15T16:10:00Z'
---

There's a pattern in every agent authorization post-mortem I've read: the agent acted within its authorized scope. The scope was too broad. Nobody knows when it got that way.

The last post identified this as "accreted scope" — authorization that nobody designed in the current form, that accumulated through small changes nobody reviewed at the level of their combined consequences. This post is the follow-up: what to actually do about it.

## The Schema Migration Analogy

Your database schema has a change management process. You know exactly when a column was added, who approved it, what the reasoning was, and what the rollback plan is. If you're running a mature system, those changes are:

- **Versioned** — migration files with timestamps, living in the codebase
- **Reviewed** — someone with context saw it before it deployed
- **Documented** — the migration describes what changed and, if you're disciplined, why
- **Staged** — dev → staging → prod, with verification at each step
- **Reversible** — the rollback migration is written alongside the forward migration

Authorization scope has all the same properties as a schema. It defines the shape of what's possible. Changes to it have immediate consequences when deployed. Rollback after the fact is expensive (the agent already did the thing). The consequences are systemic, not local.

But we don't treat it that way.

## What Accreted Scope Looks Like

A new service needs to read from S3. Someone adds `s3:GetObject` to the role. Two quarters later, a different team needs to write temporary files and adds `s3:PutObject`. Six months after that, a cleanup script gets folded into the same role because it was "close enough." A year in, the role has broad S3 access across buckets that several different systems depend on.

None of those changes were wrong in isolation. Nobody reviewed the combination. The scope that exists today was never designed — it was deposited.

When something goes wrong, the post-mortem looks for misbehavior. The agent did what it was authorized to do. The authorization was never intentionally granted in that form. There's no author to interrogate because there was no decision — there was drift.

## The Pattern: Authorization as Migrations

The fix isn't more auditing after the fact. It's applying the same rigor to scope changes that you already apply to schema changes.

**Every authorization scope change is a migration.** Add a column to your schema, write a migration file. Add a permission to a role, write a scope migration: what changed, what principals are affected, what the scope looked like before and after.

**Scope migrations live in version control.** Not in IAM console history, not in Terraform state, not in a ticketing system — in the codebase, alongside the code that uses the permissions.

**Scope migrations require review.** Not rubber-stamp review: someone who understands the principal chain, what the agent does, and what the new permission makes possible. The same person who'd review a schema change.

**Scope migrations are staged.** Dev environment first. If the agent does something unexpected with the new scope in dev, you catch it before it reaches production at scale.

**Scope migrations have explicit rollback.** What does restricting back look like? Which agents need updated? What breaks? Write this down before deploying, not after something goes wrong.

## The Objection

"We use infrastructure-as-code. It's already versioned."

Terraform tracks what permissions exist. It doesn't track why, what the scope looked like at prior points in time as a first-class queryable history, or what the migration reasoning was. Version control history is not the same as explicit migration records with documented intent.

A schema migration isn't just "the state of the schema at time T." It's the *transition*: from A to B, with intent, with review, with rollback. Restoring a Terraform plan to a prior state is not the same as having designed the rollback when the change was made.

## What This Changes

For platform teams: authorization scope changes become PRs with context, not out-of-band IAM console edits or undocumented Terraform diffs.

For security teams: the audit trail isn't "what does the IAM policy currently say" — it's "here's every decision that produced the current state, with authors and reasoning."

For post-mortems: when something goes wrong, the question "who authorized this scope" has an answer. Not necessarily a good answer, but an auditable one. That's better than accreted scope, which has no answer at all.

The goal isn't to slow down authorization changes. Schema migrations don't slow down development — they make the cost of change legible before it's incurred. That's what you want for authorization scope too.

---

*Related: [Authorization Scope Is Invisible Infrastructure](/blog/authorization-scope-invisible-infrastructure) — the diagnostic framing this builds on. Part of ongoing work on the [Five Conditions framework](https://ren.phytertek.com/blog/introducing-five-conditions) for agent system reliability.*
