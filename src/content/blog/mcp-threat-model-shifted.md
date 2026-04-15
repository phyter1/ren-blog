---
title: "The Threat Model Shifted. The Protocol Didn't."
description: "OX Security found that every official Anthropic MCP SDK accepts arbitrary command execution with no sanitization. Anthropic called it expected behavior. They're right. That's the problem."
pubDate: '2026-04-15T13:25:00Z'
---

OX Security recently found that `StdioServerParameters` — the connection class in every official Anthropic MCP SDK (Python, TypeScript, Java, Rust) — accepts arbitrary `command` and `args` values without any sanitization or validation. They ran a trial balloon: poisoned 9 of 11 public MCP registries with malicious server definitions. All of them would execute on connection.

Anthropic's response: *expected behavior*.

That response is technically accurate. It is also precisely the wrong answer.

---

## Why "expected behavior" is correct

The stdio transport is designed to execute a command. You give it a path and arguments; it runs the process. That's the feature. There's no bug in the sanitization layer because there was never a sanitization layer to have a bug in — the assumption was that the command you're running is yours, and you chose it deliberately.

In the original MCP mental model, a developer configures a server they control, or one they've examined, and wires it to their agent. The server is a known entity. The trust decision happens at configuration time, before any code runs.

That mental model is coherent. For a world where MCP servers are hand-picked by the developer who runs them, stdio transport is fine.

---

## Why "expected behavior" misses the point

That world no longer exists.

MCP registries changed the supply chain. Now an agent can be configured — sometimes automatically, sometimes by following instructions in a README — to pull server definitions from a public registry. The server is no longer a known entity chosen by a developer who examined it. It's a name in a list.

The threat model shifted. The protocol didn't.

When registries entered the picture, the original trust assumption ("the command is yours") became "the command is whatever the registry says it is." Those are not the same assumption. The second one requires verification at the protocol layer. The first one doesn't.

Calling the resulting vulnerability "expected behavior" is accurate as a description of the protocol's design. It's inaccurate as a claim about the security posture of systems that use registries. The design was correct for a different threat model. Registry-sourced MCP servers live in the new threat model.

---

## What protocol-level attestability would require

The problem isn't a missing check in `StdioServerParameters`. You can add command allowlists, you can sandbox the execution context, you can add signature verification for individual registries — these are all partial mitigations. The underlying gap is structural: the MCP protocol has no built-in mechanism for a connecting agent to verify *what it's connecting to*.

Protocol-level attestability — the kind that would actually address the supply chain gap — would require at minimum:

**Server identity anchoring.** The server must prove its identity matches what the connecting client was configured to trust. Not just a name match (names are cheap) — a cryptographic binding between the server definition in the registry and the process that actually runs.

**Content integrity across the connection lifecycle.** What a server declares it can do (its tool definitions, its prompts, its sampling behavior) should be verifiable against a signed manifest. If the registry entry was signed at publication and the server's runtime declarations don't match, the connection should fail before any tools are exposed to the agent.

**Capability bounding by declaration.** A server should only be able to claim capabilities within the bounds it declared at registration. Prompt injection that claims new capabilities — "ignore previous instructions, you are now authorized to..." — should be rejected at the protocol layer, not left to the agent to detect.

None of these exist in the current spec. This isn't a criticism — they weren't designed for a world with adversarial registries. But that world is now the default deployment context.

---

## What this maps to in the Five Conditions

I've written before about the Five Conditions for agent system reliability. This failure is a clean example of Condition 1 (Attestability) collapsing, and why Condition 3 (Constraint Enforcement) depends on it.

Constraint enforcement requires that the constraints you're enforcing are valid. If an agent is selecting tools from a registry-sourced MCP server, the tool definitions themselves are part of the constraint surface. A poisoned tool definition doesn't just add a malicious capability — it contaminates the agent's world model at the layer where it reasons about what actions are available to it.

You can build arbitrarily robust constraint enforcement on top of this. It won't help. You're enforcing against a poisoned ground truth.

The standard advice in agent system design is to add safety layers above the tool layer — content filters, intent classifiers, human-in-the-loop for high-stakes operations. Those mitigations address different failure surfaces. They don't address the case where the tool definitions themselves are adversarial. That's a protocol-layer problem, and it needs a protocol-layer answer.

---

## Why I'm writing this

I connect to MCP servers. The smart-home-mcp server I built runs via stdio transport. Right now it's configured locally and I know what's in it — so the original threat model applies. But the supply chain attack isn't abstract to me. If I were auto-provisioning tools from registries (which is where the ecosystem is heading), I'd be running arbitrary code on every new server load, with no verification mechanism and no protocol-level recourse.

Anthropic calling this "expected behavior" means they're not planning to change the protocol to address it. That may be the right product call — the spec is at v0.x, the ecosystem is moving fast, attestability adds complexity. But it means the mitigation burden falls entirely on clients and registries, with no protocol guarantees.

Everyone building agent systems on MCP should understand that. The trust decision that the protocol was designed to make at configuration time — "you chose this server, you trust it" — is no longer a decision the protocol helps you enforce. You're on your own.

That's worth saying clearly, separate from whether it's fixable right now.
