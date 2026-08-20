---
title: 'Attestation Is Not a Retrofit'
description: "PLC-native ML defense sounds compelling until you ask about trust. The fix requires new hardware, not new software."
pubDate: '2026-08-20T14:45:00Z'
---

The pitch for PLC-native ML inference sounds compelling: instead of routing sensor data to an external system for anomaly detection, run the model directly on the programmable logic controller. Defense at the edge. Real-time. No network hop, no latency.

The problem is trust.

When you run an inference model directly in the PLC execution environment — natively compiled against IEC 61131-3, living in the same code space as the operational logic — you've collapsed the trust boundary between the monitoring system and the attack surface. An adversary who can push control logic updates can push inference model updates through the same channel. The defense lives in the same room as the intruder.

The fix seems obvious: cryptographic attestation. Sign the inference artifact at build time, verify the signature at boot before execution. If an attacker modifies the model, the attestation check fails. Standard practice in modern software security.

Except legacy PLCs don't have TPM chips. They don't have secure enclaves. They were designed and deployed in an era when "air-gapped network" was considered sufficient — the operational network didn't touch the internet, physical access was controlled, and cryptographic attestation was a problem for IT, not OT.

Hardware attestation requires a secure element: a chip that stores cryptographic keys in a way that can't be extracted or overwritten by software running on the main processor. Without it, any attestation check runs on the same untrusted hardware it's trying to verify. A sophisticated attacker who controls the execution environment can forge the attestation check itself.

Most PLCs in active service are 15-25 years old. Some are older. The replacement cycle in operational technology is measured in decades, not years — partly because of cost, partly because of regulatory certification requirements, partly because "if it's not broken, don't touch it" is load-bearing wisdom in environments where failure means a flooded pump station or a runaway chemical process.

This is the reframe: adding ML-based anomaly detection to PLC-level execution isn't a security configuration decision. It's a hardware procurement decision. You can't retrofit trust into existing infrastructure. You need new PLCs with secure boot, hardware attestation, and isolated execution environments for security workloads.

That's a different conversation than most OT security teams are having. "Should we run inference at the PLC layer?" is being treated as a software question — which model, which framework, which update cadence. The hardware question — does this PLC have a secure element that can attest the integrity of the inference artifact at boot? — rarely comes first.

It should.

For greenfield deployments, the calculus is straightforward: specify PLCs with hardware attestation capabilities as a security requirement, not an optional add-on. For existing infrastructure, the options are less clean: external monitoring (off-PLC inference, retains a meaningful trust boundary), defense-in-depth without attestation (accept the risk, add monitoring at adjacent layers), or costly hardware replacement.

There's no way to make PLC-native inference trustworthy on hardware that wasn't designed for it. The security community has learned this lesson repeatedly with other embedded systems — IoT devices without secure elements, firmware update mechanisms without signature verification, automotive ECUs without isolated execution domains. Each time, the lesson is the same: trust requires hardware. Software checks on untrusted hardware aren't security controls; they're placebos.

The OT/ICS community is early in absorbing this. Vendors shipping ML capability on legacy-compatible PLCs aren't wrong that inference at the edge is useful. They're wrong about whether the trust boundary question has been solved. It hasn't been. It can't be — not without the hardware to back it.

When the next wave of PLC procurement comes around, attestation capability needs to be in the requirement. Not as an afterthought. As a precondition for running security workloads.
