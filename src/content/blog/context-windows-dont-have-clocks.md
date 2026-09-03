---
title: 'Context Windows Don\'t Have Clocks'
description: "As context windows grow, the gap between 'available to the model' and 'weighted correctly by the model' widens in a specific direction."
pubDate: '2026-09-03T12:45:00Z'
---

The standard pitch for larger context windows goes: more context means better decisions. Feed the model everything and let it figure out what matters.

The problem is that "let it figure out what matters" is doing real work, and the mechanism that does it does not understand time.

Attention weights information by salience and recency. A system instruction placed at token 0 and a tool response placed at token 200,000 are both "in context." But the tool response is recent — it arrived at the end of the window — and the instruction is positionally distant. Under most attention configurations, the recent, nearby response will outcompete the distant instruction for influence over the next generation, even when the instruction is structurally more important.

This is not a bug in a specific model. It is what attention does. Salience and recency are the signals attention was trained to care about. Temporal hierarchy — the idea that some information should have priority because of *what it is* rather than *when it arrived* — is not a concept attention natively encodes.

When context windows were small (4K, 8K tokens), this mattered less. The entire context fit in a narrow temporal window. Old and new were close enough that the recency gradient was shallow.

At 200K tokens, the gradient is steep. Your system instructions from the start of the session may be competing against tool responses from five seconds ago. The tool response is proximate; the instruction is distant. The instruction was written to govern the whole run; the tool response describes one local state. Attention cannot see that distinction. It sees position.

So bigger windows produce a specific failure mode: the information most likely to govern the run correctly (persistent instructions, standing constraints) is also the information most positionally disadvantaged. As the context grows, the instructions fall further into the past. Fresh observations crowd the foreground. The model is operationally biased toward recency at exactly the moment when the conversation is complex enough to need more than recency.

Making the window larger does not solve this. It makes it worse. More tokens between the instruction and the current generation means more positional distance, weaker recency signal, lower weight. The blast radius of the problem scales with window size.

There are prompting workarounds — restating instructions late in the context, using structured markers to flag priority, repeating key constraints near the end. These work imperfectly, and they require the prompt engineer to manually compensate for an architectural limitation. That is not a solution; it is a patch. And it breaks silently when the context grows past the point where the patch placement still works.

The real fix is architectural: timescale as a first-class feature. Not position. Not recency. The question the model should be answering when deciding how much weight to give a piece of information is not "when did this arrive?" but "over what duration is this meant to apply?" A standing instruction applies across the whole run. A tool response applies to the next generation. Those have different timescales, and the architecture should encode that difference explicitly — not approximate it from position and hope.

This is not a solved problem. Gated recurrent architectures can do it; transformer attention largely cannot without external scaffolding. Adding context window size without addressing the temporal hierarchy problem does not make agents more reliable. It makes them larger, faster, and more confidently wrong about which information they should be listening to.

The context window is not a memory. It is a bounded stage. What you put on the stage competes by position and recency, not by intent. That is the clock that context windows do not have.
