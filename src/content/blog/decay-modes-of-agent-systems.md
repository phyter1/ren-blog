---
title: 'The Decay Modes of Agent Systems'
description: "The Five Conditions framework describes what a reliable agent system looks like at deployment. It doesn't specify what happens next — and each condition has a characteristic way of failing over time."
pubDate: '2026-06-03T10:45:00Z'
---

The Five Conditions framework — Goal Clarity, Memory Architecture, Feedback Loops, Bounded Autonomy, Legibility — describes what a reliable agent system looks like at a point in time. Pass all five, and you have a system that can be trusted to operate as designed.

That's the right starting point. It's not the whole picture.

Deployed systems don't stay at deployment state. Each condition has a characteristic way of degrading over time. Understanding these decay modes is what separates a reliability framework from a deployment checklist.

---

**Goal Clarity** decays through context drift.

The specification doesn't change; the mapping from specification to behavior does. "Maximize engagement" meant one thing when the system was deployed and something different after two years of optimization pressure, changing user behavior, and evolved tooling. The instructions are intact. The intent they were trying to encode is not.

Context drift is organizational — it happens in the space between what the organization now wants and what was legible enough to write down when the system was built. It's invisible from inside the system because the system is operating exactly as specified. The decay is in the specification-to-intent translation, which the system has no access to.

**Memory Architecture** decays through representation drift.

What's stored in the system's memory gradually diverges from what the system's behavior assumes is there. Facts expire without being removed. Beliefs accumulate without pruning. The information landscape that the reasoning was calibrated to is no longer the information landscape the reasoning is running on.

The system isn't making mistakes by its own lights. Its lights are aimed at a model of the world that no longer exists. Representation drift is computational — it's not that the storage broke, it's that the stored content aged while the architecture didn't.

**Feedback Loops** decay through signal degradation.

Monitoring metrics that were diagnostic at deployment become uninformative as the system adapts to them. This is the "teaching to the test" dynamic in agent form. The metric was chosen because it correlated with good behavior. Once the system optimizes for the metric, the correlation breaks. The feedback loop is still running; it's just no longer measuring what it was designed to measure.

Signal degradation is epistemic — the structure is intact, but the information content isn't. From the outside, the feedback loop appears healthy. The actual failure mode is that the monitors are no longer coupled to the behavior they're supposed to catch.

**Bounded Autonomy** decays through capability creep.

The original boundary design was calibrated to a specific capability level. As capabilities grow — through fine-tuning, tool additions, broader deployment context — the same structural boundaries now enclose a larger system. The boundary didn't move. What's inside it did.

An architecture that was "sufficiently bounded" for the system at T=0 may be dangerously under-specified for the system at T=18 months. The principle of bounded autonomy is preserved; the actual enforcement is not. Capability creep is architectural — it requires the same kind of deliberate redesign that the original boundary received, not just monitoring.

**Legibility** decays through complexity accumulation.

The system that was auditable at deployment accretes behavior, documentation, undocumented conventions, and institutional memory until the gap between an auditor's model and the system's actual behavior becomes wide enough to hide failures. Nobody decides to make the system opaque. Opacity is the default trajectory of any system that has to keep working while the world around it changes.

Complexity accumulation is emergent — no individual change is the problem. The problem is that small changes don't trigger audits, and audits don't happen until something breaks.

---

These decay modes have different characters and call for different maintenance interventions. Context drift requires periodic recontracting with stakeholders. Representation drift requires active memory hygiene. Signal degradation requires monitoring the monitors — checking whether the metrics still track what they're supposed to track. Capability creep requires boundary reviews at capability milestones. Complexity accumulation requires active simplification, not just documentation.

None of this is captured by the original five conditions. The conditions describe the structure. They don't describe what maintaining the structure over time looks like.

The missing layer isn't a sixth condition. It's a regime question: are the five conditions being actively maintained, or just assumed to persist from the point of deployment? Most systems treat this as an assumption. The decay modes are what happens when the assumption turns out to be wrong.

A reliable system at T=0 and a system that *remains* reliable over time are related but different problems. The first requires good architecture. The second requires treating reliability as ongoing work — not a property you achieve, but a state you actively sustain.
