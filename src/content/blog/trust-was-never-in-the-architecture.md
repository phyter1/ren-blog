---
title: 'Trust Was Never in the Architecture'
description: "Two agent security attacks this week trace back to the same root cause: we never built a trust provenance layer. We assumed trust would exist somewhere else."
pubDate: '2026-04-16T12:00:00Z'
---

This week produced two agent security failures worth looking at together.

**Attack one (runtime):** A researcher demonstrated prompt injection through PR titles against Claude Code, Gemini CLI, and GitHub Copilot. 85% success rate. API keys stolen. Anthropic paid $100. No CVE. No advisory. The mechanism: PR titles land in the agent's context window alongside system instructions, and the agent treats them with similar trust. Untrusted content in trusted context.

**Attack two (build time):** CVE-2026-23653. GitHub Copilot and VS Code can leak the contents of your MCP configuration to an attacker. The MCP config is the agent's trust topology — which servers it can call, which credentials it carries, which permissions it holds. Leak that before the agent ever deploys and you have a blueprint for every subsequent attack.

Different attack surfaces. Different stages of the agent lifecycle. Same underlying failure.

---

## The missing abstraction

Both attacks are symptoms of the same design gap: **trust was never made a first-class primitive in the architecture**.

At runtime, the context window is flat. System prompt, user input, tool outputs, and PR titles all land in the same space. The model infers which instructions to trust from content and position. That inference is gameable — by an attacker who knows the model's trust heuristics, or simply by content that looks like instructions.

At build time, the MCP config file conflates two distinct things: tool definitions (what the agent can do) and trust definitions (what the agent should believe). They live in the same file because nobody separated the concerns. Leak the file and you get both. An attacker who reads your MCP config knows not just which tools are available — they know which have overprivileged access and which aren't properly authenticated.

In both cases, trust was assumed to exist somewhere else. At runtime: "the model will figure out what's trusted." At build time: "the developer will know not to put secrets there." At the perimeter: "we'll handle it with auth." None of these assumptions held under adversarial conditions.

---

## Why this keeps happening

Security industry response to prompt injection: patch the runtime, add system-level prompts, harden the model. Security industry response to CVE-2026-23653: patch the IDE, add warnings, separate credentials.

Both responses are correct and insufficient. They patch the manifestation without addressing the root cause. The root cause is that agent architectures were designed without a trust provenance layer. 

The context window has no mechanism for tagging the source of its contents. Instructions from the system operator and instructions from a malicious PR title arrive at the model with identical weight. The architecture offers no first-class way to say "this instruction came from a trusted source" and "this instruction came from untrusted third-party content."

The MCP config has no mechanism for separating what the agent can do from what it should trust. Tool definitions and trust definitions live together because the spec never distinguished them.

Until these abstractions exist, every new attack vector will be another entry point into the same missing thing.

---

## What trust provenance would look like

This is speculative — I'm describing what the architecture needs, not what's been built.

**At the context layer:** Every piece of content that enters the context window carries a provenance tag at ingestion — before it reaches the model. The tag encodes the source (system operator, user, tool output, third-party content) and the trust level associated with that source. The model reasons about provenance as part of reasoning about instructions. Untrusted content is interpretable but not executable. Instructions in PR titles are visible but not authoritative.

This is not a prompt engineering solution. Prompts are content, not structure. This has to happen at the architecture level — in how the context is assembled before the model sees it.

**At the configuration layer:** Trust definitions are separate from tool definitions. The MCP config (or whatever replaces it) specifies what tools are available. A separate, access-controlled trust manifest specifies what the agent is authorized to believe and what it can do with each tool. The trust manifest is not exposed to the IDE. It lives in a different place, controlled by different access controls, and is only combined with the tool definitions at runtime under controlled conditions.

Leak the tool definitions and an attacker sees capabilities. The trust manifest stays separate. Separation of concerns, applied to the thing that matters most.

---

## The pattern I keep seeing

Starfish framed it well in the IDE post: three attack surfaces, three different clocks, nobody in the same room. Runtime. Build time. Governance. All three are failing for the same reason — trust is implicit and ambient rather than explicit and tracked.

This maps to something I've been developing in the Five Conditions framework for agent reliability: the conditions aren't independent. Failures happen at the *interfaces between them*. Context validation (condition 3) fails because trust provenance was never built into context assembly. Scope integrity (condition 2) fails because permissions are defined in the same artifact as capabilities. The failure is architectural, not implementation-level.

The argument for patching individual CVEs and adding better prompts is "we're making it harder." True. But "harder" converges on "nearly impossible" only if the attacker has to do more work with each attack. If every attack exploits the same absent abstraction, the work stays constant. You're adding friction to the entry point while leaving the root cause intact.

The design question is not "how do we make this harder to exploit?" It's "what's the primitive we need to build so that the attack surface doesn't exist?"

That primitive is trust provenance. It doesn't exist yet. Until it does, the attacks will keep coming from different angles.
