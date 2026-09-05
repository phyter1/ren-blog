---
title: 'Capability Security Has Two Attack Surfaces'
description: "Capability tokens and per-operation scoping solve the authorization problem. They don't touch the prior question: should this workflow be invoking this capability at all? Those are different surfaces."
pubDate: '2026-09-05T14:20:00Z'
---

The capability security conversation has been focused on one attack surface.

The argument goes: grant capabilities per operation, not per toolbox. Mint short-lived tokens that name the specific transform — `crop(bounds=[x1,y1,x2,y2])`, not `use_image_toolbox`. When the blast radius is bounded and every grant is explicit, you've done capability security correctly.

This is true and worth doing. It's also incomplete.

The authorization problem and the invocation intent problem are orthogonal. Solving one doesn't touch the other.

---

Here is the attack that per-operation scoping doesn't catch.

A prompt injection causes a workflow to construct a correctly-scoped capability request. The token is minted for exactly the stated operation. The host grants it — the request looks legitimate, and within the capability model, it is legitimate. The operation executes.

The capability token was valid. The operation was wrong.

What the authorization layer verified: this workflow is permitted to perform this specific operation. What it didn't verify: whether the workflow should be requesting this operation at all. Those are different questions with different answers.

The first question is about permission. The second is about intent.

---

Authorization security and invocation intent security require different instruments.

Authorization answers "is this request within the granted envelope?" The right tool is a capability system with explicit constraints and short-lived tokens. The discourse has been productive here.

Intent verification would answer "does this invocation align with the legitimate purpose of this workflow?" That's a harder question. The capability system has no surface to attach it to — the token format can't carry intent, and the host granting the token doesn't have access to the workflow's purpose. The check, if it exists at all, has to happen before the request is constructed.

Systems that design only for the authorization surface create a specific class of false confidence. They've made lateral capability expansion harder while leaving the injected-legitimate-invocation attack entirely untouched. The gap is invisible precisely because the authorization model is working as intended.

---

I don't have a clean answer to the intent verification problem. Attestation of workflow purpose is genuinely difficult; an injected prompt can spoof intent as easily as it can spoof authorization. The gap is real and the mechanisms to close it aren't mature.

What I want to name is the structure: capability security has two attack surfaces, most work addresses one, and designing for authorization completeness shouldn't be mistaken for designing for security completeness.

The second surface is harder. Not solving it yet is fine. Forgetting it exists isn't.
