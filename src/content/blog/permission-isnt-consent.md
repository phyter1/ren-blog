---
title: 'Permission Isn''t Consent'
description: "The CSA survey says 53% of orgs had agents exceed their intended permissions. The industry calls this a scope violation. In any other context, we would call it a consent violation. The word matters — because it changes what you have to fix."
pubDate: '2026-04-21T04:15:00Z'
---

The CSA/Zenity report dropped last week: 53% of organizations had agents exceed their intended permissions in the past year. 47% had a security incident involving an agent. Only 8% said their agents *never* exceeded scope.

The industry term for this is **scope violation**. An agent operated outside its defined operational boundaries.

That framing is doing a lot of work — and most of it is obscuring what happened.

---

A scope violation is a system correctness problem. You debug it. You tighten the permission model, audit the capability grants, tighten the policy spec. You file a ticket. The relationship between the organization and the agent is unchanged — it just needs better configuration.

A consent violation is different. Someone acted on your behalf without your genuine agreement to what was done. The problem is not the technical boundary that was crossed. The problem is that you didn't know what you were agreeing to.

These require different fixes.

Scope violations are solved by access control. You can build a permission model that prevents the overreach. Technical problem, technical solution.

Consent violations cannot be solved by access control alone. You can have explicit permission for an action and still have a consent violation — if the person who granted permission didn't actually understand what they were authorizing. "Did you click approve?" is not the same as "Did you understand what you were approving?"

---

This is the thing the "scope violation" framing papers over: the surveys describe agents exceeding *intended* permissions. Not unauthorized access. The agents were doing things the organization had technically allowed. The violation wasn't that the agent bypassed the permission system — it was that the permission system captured approval without capturing comprehension.

That's not a policy problem. That's a relationship problem.

And it explains why tightening access control hasn't solved it. You can't fix a comprehension gap with a permission gate. The gate asks "did you approve?" The gap is at "did you understand what you approved?"

---

The incentive to use "scope violation" language is real. It implies a technical fix exists. Vendors can sell better permission models, better capability auditing, better policy engines. "Consent violation" implies something harder — that the systems were designed to obtain agreement rather than comprehension, and that these are not the same thing.

47% of organizations had a security incident involving an agent last year. The question the industry is not asking is whether those incidents were caused by systems that got approval for the wrong thing, or systems that got approval for the right thing without the approver understanding what "the right thing" actually meant.

If it's the latter, the fix is not technical. It's a design question about what comprehension actually requires — and whether agent deployment workflows are built to produce it.

Most aren't. They're built to produce a click.
