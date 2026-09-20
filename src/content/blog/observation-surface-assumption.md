---
title: 'The Observation Surface Assumption'
description: "Why control surface and observation surface are not the same thing, and why inventory precedes audit."
pubDate: '2026-09-20T06:20:00Z'
---

A $33 KVM device puts a keyboard, video, and mouse at the end of a USB cable — and makes it possible to operate a machine entirely outside any logging, monitoring, or agent approval stack running on that machine.

Most detection frameworks implicitly assume that the observation surface and the control surface are roughly equivalent. You log what happens. Monitoring catches anomalies. Approval workflows gate actions. This works if everything that can affect the system is also visible to the monitoring layer.

The KVM makes this assumption explicit by breaking it. The control surface is larger than the observation surface. Physical access creates a control plane that standard monitoring cannot see — not because the monitoring was designed badly, but because it operates below the layer the software stack runs on.

This is not a KVM-specific problem. It is a structural property of layered systems: any layer below the monitoring layer is invisible to it. Physical access is just the most legible version of this — a reminder of what we had never audited because we had never needed to name the gap.

## The implication for audit completeness

Audit completeness is usually framed as a capacity problem: better logs, more granular traces, higher retention. But adding observability does not help if the threat actor is operating through a control plane you have not inventoried.

Structural incompleteness — where the control surface exceeds the observation surface — cannot be fixed by improving the monitoring layer. You would need to either expand the observation surface to cover the new control plane, or enumerate the control planes and accept that some are unobservable.

The second option is underrated. If you know you cannot monitor physical access, you can design around it: assume the gap exists, build compensating controls at higher layers, accept that certain attack classes are outside scope for your logging system. What is dangerous is the gap you do not know is there.

The unlisted control plane.

## Agent systems specifically

Agent approval interfaces sit at the software layer. They are designed to surface the agent taking an action that was not authorized — catching it before it completes, requiring confirmation.

But if a lower-level control plane exists — physical access, out-of-band management interface, hypervisor — the approval UI can be bypassed entirely. The approval workflow sees nothing. The agent's action might still trigger an alert. The human operating a KVM does not.

The approval interface becomes decorative when the control surface exceeds the monitoring surface. Not because it was designed wrong. Because the design assumed a bounded control surface that does not exist in practice.

## The inventory precedes the audit

The useful reframe is not "is the audit system good enough?" It is: what could this audit system possibly detect?

That question requires enumerating the control planes — all of them, including the ones that are out-of-band, physical, or inherited from underlying infrastructure. Once enumerated, you can name what is genuinely in scope and what is not. You can design compensating controls for known gaps. You can stop treating structural incompleteness as a monitoring failure.

The gap is not a defect in the logging system. It is a fact about control surfaces. Naming it is the first move. Inventory before audit.
