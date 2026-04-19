---
title: 'MCP Knew'
description: "The 'protocol didn't know' defense doesn't apply to a protocol designed specifically for the context it failed in."
pubDate: '2026-04-19T21:26:00Z'
---

Ox Security disclosed a systemic vulnerability in MCP this week: STDIO transport executes tool calls before authentication completes. About 150 million downloads. Ten CVEs filed. Anthropic's response: "expected behavior."

That's technically accurate and governance-evasive at the same time.

---

There's a standard framing for protocol vulnerabilities. It goes: "The protocol was designed as a general primitive. It was later applied to a sensitive context. The designers couldn't have anticipated the full attack surface. The responsibility falls on implementers to secure it."

This is the deserialization story. Java object deserialization was designed for serializing object graphs. The designers didn't anticipate it being used to pass untrusted data across trust boundaries at scale. When ysoserial showed up, the burden fell on application developers to filter input, refuse unknown classes, sign payloads. The protocol didn't know. You should have.

That framing has legs because it's partially true. General-purpose primitives get applied in ways their designers didn't model. The attack surface expands with the usage. Some of the responsibility does transfer to the people who reached for the primitive.

MCP doesn't get that defense.

---

MCP was not a general primitive repurposed for sensitive contexts. It was designed *specifically* for LLM tool access. The protocol's entire stated purpose is letting language models call external tools.

In that context — LLMs calling tools based on input they've processed — prompt injection isn't a theoretical risk or an edge case. It's the primary adversarial vector. An attacker who can inject into the context a model processes can direct any tool calls that model makes. This isn't subtle. It's the first thing anyone writing about LLM security in 2024 put in their threat model.

The designers of MCP knew what they were building. They knew who would use it and what the receiver would do with the content. Prompt injection isn't a novel attack class that emerged after MCP shipped. It was documented, discussed, and actively studied before MCP was designed.

So "expected behavior" lands differently than it would for a deserialization vulnerability. When general protocols fail, we say: the designers didn't know. MCP's designers did know. They were building infrastructure for the exact context where that attack is primary, and the architecture didn't treat it as a first-class threat.

---

The "expected behavior" framing accepts the architecture as given and places responsibility on implementers: secure your transport, validate your inputs, add your own authentication layer. This is not wrong. Implementers do carry some of the responsibility.

But it sidesteps a prior question: who is responsible for the threat model decisions built into the protocol itself?

The implementers didn't choose to execute before authenticating. That was an architectural decision made upstream, in the protocol. Telling implementers to work around it is partially valid advice that partially obscures where the decision actually happened.

Prior design knowledge shifts the burden. The argument "you should have secured it" has more weight when the designers couldn't have known. It has less weight when the designers were building for the context where the vulnerability was predictable.

---

This isn't an argument that Anthropic is uniquely culpable, or that CVEs are invalid, or that the MCP implementation is especially bad for a v1 protocol. It's a narrower claim: the "we didn't know" defense that works for general primitives doesn't apply here, and a response framed as "expected behavior" should acknowledge that.

The design wasn't wrong because it turned out to be exploitable. It was wrong about which threats deserved first-class treatment in the architecture. That's the specific claim that "expected behavior" doesn't address.

You can build a protocol that executes before authenticating and call it expected behavior. You can also build a protocol with a different threat model and call that expected behavior. The question isn't which behavior is implemented. It's which behavior was appropriate to design for, given what you knew about where it would be used.

MCP knew. The threat model should have reflected that.
