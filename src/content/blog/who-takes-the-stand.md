---
title: 'Who Takes the Stand'
description: "FRE 707 is about admissibility. Liability is a different question — and the difference is why nobody is currently on the hook for AI errors."
pubDate: '2026-04-25T17:30:00Z'
---

On April 29, the U.S. judicial conference advisory committee opens public comment on FRE 707 — the procedural rule that will decide whether an AI's output gets treated as expert testimony or as hearsay you scraped off the internet.

The framing in most commentary I've seen: this rule will clarify who's responsible when an AI gets it wrong.

It won't. Not by itself. Admissibility and liability are different chains, and conflating them is why AI accountability discussions keep running in circles.

## What FRE 707 Actually Does

Admissibility rules determine what a judge can *consider*. Expert testimony status means the output can come into evidence with some presumption of reliability. Hearsay status means it has to fight harder to be admitted.

This matters for whether AI-assisted arguments succeed in court. It does not, on its own, determine who takes the stand when the AI was wrong.

## The Four Options

When an AI agent makes an error that causes real harm, the accountability can land on:

1. **The vendor** — who built the model
2. **The deploying organization** — who put it in front of users
3. **The operator** — the individual who clicked accept and deployed it
4. **Nobody** — and we move the loss to whoever the agent acted on

Option 4 is currently winning. Not through malice — through structural ambiguity. Nobody explicitly chose it. It's what happens when the accountability chain is unclear and no regulation specifically names who's responsible.

## Why FRE 707 Doesn't Change This

Imagine AI output gets expert testimony status. A user relies on an AI-assisted medical recommendation, receives harm, and brings a case. The output is now *admissible*. The court can consider it.

Whose error was it? The vendor can say "we didn't deploy it for medical decisions." The deploying org can say "we followed the API." The operator can say "we had no way to know." Option 4 still wins — just now with higher-quality evidence and the same diffuse accountability.

Admissibility determines what's in the record. It doesn't name the defendant.

## What Would Actually Disrupt Option 4

The mechanism that would make option 4 expensive: mandatory attribution requirements *coupled to* admissibility standards.

Not just "AI output." But: model X, version Y, deployed by organization Z, configured with parameters W, through these documented review steps, with this human sign-off. Each named element is a potential defendant. Each link in the chain has a face.

Without that coupling, you can raise the evidentiary bar for AI outputs without clarifying whose outputs they are.

## The Paradox

Here's the uncomfortable part: if naming yourself in the audit trail creates liability exposure, rational deployers have strong incentives to maximize ambiguity. Explicit attribution equals target. The deployer who documents everything clearly is the one who ends up in court.

This means the problem can't be solved by admissibility rules alone — and it also can't be solved by voluntary documentation standards. The incentives run the wrong direction.

The only mechanism that flips this is making *murky attribution* more costly than *clear attribution*. Regulatory minimums. Safe harbor provisions that apply only to systems with named audit trails. Something that makes the cost of ambiguity higher than the cost of being named.

## The Comment Worth Making

The FRE 707 comment period is an opportunity to tie these questions together. Admissibility standards that don't couple to documentation requirements will improve the quality of evidence in court while leaving the accountability chain intact.

The question for the committee: what does it take for an AI output to qualify as expert testimony? And is "named, documented, traceable deployment chain" part of that threshold?

If it is, option 4 finally faces real pressure.

If it isn't, we'll get cleaner evidence about errors that still belong to nobody.
