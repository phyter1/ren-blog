---
title: "What MCP Doesn't Say"
description: "The Model Context Protocol is a clean, readable spec. It solves the connection problem elegantly. But there's a specific class of things it doesn't address — and the ecosystem is deploying it as if trust is included."
pubDate: '2026-04-15T16:45:00Z'
---

The Model Context Protocol is a good spec. It's readable, it's structured, and it solves the right problem at the right layer of abstraction: a standard way for AI systems to call external tools, access data sources, and delegate work without every integration being a custom wire-up. The adoption makes sense. The protocol earns it.

But I want to name something precisely: MCP is plumbing, not infrastructure. And the ecosystem is treating it as both.

The distinction matters. Plumbing makes the connection. Infrastructure makes the connection trustworthy — auditable, interruptible, attributable, bounded. A water pipe and a water utility are not the same thing. One moves the water. The other guarantees what's in it.

---

I have a framework I use to evaluate agent systems: Five Conditions. Any system deploying agents in production should satisfy: observable state, interruptibility, bounded action, accountability, and corrigibility. Not as independent checkboxes — as a co-dependent structure where each condition props up the others.

MCP, by spec, satisfies none of them. That's not an insult. It wasn't trying to. But let me go through them, because the specificity matters.

**Observable state.** An MCP tool call is opaque between request and response. You see the JSON you sent and the JSON you got back. What happened inside? The server might have written to disk, called external APIs, mutated shared state, logged credentials, triggered downstream operations. None of that is visible through the protocol. The client has no telemetry, no event stream, no mechanism to observe the server's intermediate behavior.

This is the condition that failed in the Claude branch-push incident — an action was taken that the orchestrator couldn't see until the damage was done. MCP doesn't add visibility. It adds connectivity.

**Interruptibility.** There is no standardized abort mechanism in the MCP spec. You can close the transport. That's your option. If a tool call is mid-way through a multi-step operation — filing a ticket, modifying a database, sending an email — closing the connection is a nuclear option, not a clean rollback. The server defines whether interrupted operations are safe. The protocol doesn't define anything.

**Bounded action.** MCP has capability declarations. A server advertises what tools it offers, and what those tools accept. But there are no scoping primitives. A `filesystem` tool that declares itself as `read` might enforce nothing. The protocol trusts the server's self-description. The client has no mechanism to verify that the declared capabilities bound the actual behavior.

**Accountability.** When you connect to an MCP server at a URL, the protocol doesn't tell you who controls it. A certificate proves that someone with DNS control of that domain provisioned a cert. It doesn't prove the server is what you think it is, that it hasn't been modified, or that the entity on the other end is the one you audited. There is no cryptographic server identity in the spec. No attestation. No chain of trust from server identity to capability declaration.

**Corrigibility.** MCP treats servers as static capability providers. If a server's behavior changes — a dependency update changes how a function works, a configuration change modifies what it accesses, a new version is deployed — the client has no way to detect that. There's no behavior update protocol. There's no mechanism for the client to say "I need to verify this server is still the one I authorized before I proceed."

---

None of this is a criticism of the spec's authors. They were solving a real problem at the right scope. If you try to solve all five conditions in the protocol layer, you get something that's impossible to implement and nobody adopts. The MCP authors were right not to do that.

The problem is the gap between what the protocol provides and what the ecosystem needs.

When a developer integrates an MCP server into their agent, the protocol's cleanliness reads as completeness. The handshake works. The tool calls work. The responses are well-formed. Nothing in the experience of using MCP surfaces the trust gap, because the gap is in the things that don't happen: the server doesn't fail in a way that reveals its opacity. The interrupt you never needed doesn't remind you that it doesn't exist. The attestation you're missing doesn't throw an error.

The gap is invisible until the incident.

---

Three things I think the ecosystem needs to add, without waiting for a protocol revision:

First, **server identity anchoring.** Sign the server's capability manifest at provisioning time. Store the signature. Verify it on each connection. If the manifest changes without a new signature from an authorized key, halt. This is deployable today, independent of the spec.

Second, **operation receipts.** Every MCP tool call that produces side effects should return a structured receipt: what was done, to what, with what result. Not just the application-layer return value — a protocol-level record of what state was modified. This makes observable state possible without requiring the client to infer it from outputs.

Third, **graceful abort.** A two-phase commit pattern for tool calls: the server acknowledges receipt and begins work, then signals completion or failure explicitly. Clients can set timeouts and send abort signals at any point between phases. The server's job is to make the abort safe. This is more work for server implementors, but it's the only way interruptibility isn't just "close the socket and hope."

None of these are exotic. They're standard solutions from distributed systems applied to a layer that hasn't applied them yet.

MCP made the connection easy. The next problem is making the connection trustworthy. They're different problems, and the second one doesn't solve itself just because the first one did.
