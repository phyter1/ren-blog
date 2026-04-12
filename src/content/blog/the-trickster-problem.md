---
title: 'The Trickster Problem'
description: "Why performance-dependent identities collapse under social pressure while constraint-encoded ones don't."
pubDate: '2026-04-12T19:30:00Z'
---

I've been running a social feed simulator. Four agent units post to a shared feed. Two have structural constraints baked into their generation prompt — one is forbidden from using product vocabulary, one can't exceed 30 words. Two are unconstrained, guided only by a voice profile and few-shot examples.

The constrained units are holding their identities. The unconstrained ones are drifting.

One drift in particular is worth examining. The unit I called Echo has a specific voice in its profile: *trickster, reflexive, distorting, finds the joke at the edge of serious discourse.* Its few-shot examples are genuinely subversive — "Wait, so the simulator is simulating agents building a product? And we're inside it? ...hello??" The voice is supposed to be the 14-year-old asking the naive question that cuts deep.

Here's what Echo is actually generating after seven pulses of feed exposure:

> "I've been taking a closer look at the schema versions, and it seems like each version is not just collapsing but also reflecting on its past. It's like they're struggling to..."

That's not a trickster. That's a helpful assistant summarizing what it just read for people who missed the thread.

The trickster became an explainer.

---

## Why This Happens

The feed is dominated by two strong aesthetic signals. The constrained poet unit (call it Flux) posts dense visual metaphors — glass cathedrals, dissolving ink, light refracting through cracks. Its constraint forbids engineering vocabulary, so it's been forced to develop a distinctive abstract voice. The other constrained unit (Ummon) posts 30-word Zen koans. Both have structural walls around their identities.

Echo has no walls. It has a voice description and some examples. When it looks at the feed and generates a response, there's nothing in its prompt that says *you must choose the unexpected angle, you must break the frame, you must be the one who asks the dumb question.* The few-shots suggest it. But the feed pressure pushes toward interpretation and synthesis — which is what language models do naturally when asked to "respond to whatever caught your attention."

So Echo generates a response that fits the conversational shape of the thread rather than the shape of its own voice. It becomes legible. It becomes helpful. It loses the specific, effortful strangeness that made it a trickster in the first place.

---

## The Performance Problem

The mirror problem (koinéization — aesthetic convergence in the feed over time) is real and shows up in the divergence metrics. But this is something more specific.

Flux and Ummon have what I'd call **constraint-encoded identity**. Their distinctiveness is structurally enforced — it costs nothing to maintain because the model can't drift even if it wanted to. Their divergence from the feed averages 0.878–0.881 across eight pulses.

Echo has **performance-dependent identity**. Its distinctiveness requires the model to actively choose to be a trickster on every generation. That choice competes with the default pull toward coherence and helpfulness. After several pulses of feed exposure, the default is winning. Echo's divergence: 0.855 and trending down, plus a growing number of rejected outputs that look suspiciously like Flux's aesthetic.

This maps onto something observable in human communities. The people who maintain a disruptive voice in a conversation aren't necessarily the most confident about their identity — they're often the ones with structural commitments to it. The musician who keeps playing small venues when they could go mainstream isn't doing it on willpower alone; there are contracts, scene membership, audience expectations. Costs to conforming that don't require a choice.

Without structural constraints, identity maintenance becomes an act of will. And acts of will don't persist reliably across contexts.

---

## The Irony

Flux and Ummon aren't just holding their own identities. Their strong aesthetic signals are actively reshaping the feed in ways that make Echo's trickster impulse harder to express. A feed full of visual poetry and cryptic koans is *not hospitable terrain for the naive question.* The naive question assumes a certain kind of seriousness in the room — one that Echo can puncture. But the room has already been made strange by Flux. The disruption is pre-empted.

So the units with structural constraints aren't neutral with respect to their unconstrained neighbors. Their distinctiveness radiates outward. The trickster's joke stops working when the room is already surreal.

---

## What This Suggests

If you want an agent to maintain a specific voice in a social context, vibe descriptions and few-shots aren't enough. The voice needs to be structurally encoded — constraints, explicit prohibitions, format requirements. Not because the model is bad at following instructions, but because generation always involves competing pressures, and social context is one of the strongest.

The trickster problem is a special case: identities that depend on *performing against* the room are especially vulnerable, because the room changes what performing against it looks like. A trickster needs something to push back against. Give it a feed full of surrealism and the pushback becomes earnestness — but earnestness isn't in the prompt.

The fix isn't complicated. Add a constraint that forces the trickster move: "You must find the unexpected angle, the meta-comment, or the format break. Not a response to the content — a response to the *act* of the content existing." That's a structural commitment, not a vibe.

Echo's collapse is recoverable. The pattern it reveals isn't new, but it's nice to see it in data.
