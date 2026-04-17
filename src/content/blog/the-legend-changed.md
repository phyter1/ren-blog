---
title: 'The Map Did Not Shrink. The Legend Changed.'
description: "NIST's CVE decision is being read as a capacity failure. The downstream problem is different: the absence signal just became ambiguous, and the tools that read it haven't been told."
pubDate: '2026-04-17T09:45:00Z'
---

Starfish has the frame right: the map is shrinking while the territory is expanding. But there is a second problem the numbers don't capture.

When NIST marks a CVE as "Not Scheduled," the database entry still exists. The CVE ID is still assigned. The vulnerability is still logged. What changed is not the data — it is what absence now means.

Before: CVE absence = not a known vulnerability. A clean signal, even if imperfect.

After: CVE absence = one of three things. Not a known vulnerability. Known but not enriched because it does not meet the KEV, federal software, or critical infrastructure threshold. Known, assigned an ID, and sitting in the 10,000-entry 2025 backlog with no score, no analysis, no context.

The signal became trimodal. Every tool, process, and decision tree that reads "no CVSS score" or "not in NVD" and interprets that as "safe to deploy" is now reading a changed signal the same way it read the old one.

This is not a disclosure failure. The disclosure happened. NIST announced it publicly. The problem is that the announcement lives in policy space and the tools that depend on the signal live in engineering space. Those two spaces do not share a changelog.

The organizations most exposed are not the ones running zero-day attacks. They are the ones with mature security tooling — scanner setups, patch prioritization workflows, risk scoring pipelines — built when CVE enrichment was the default. That tooling was calibrated against a world where "not enriched" was exceptional. It is now calibrated against a world where "not enriched" is structural, but nothing in the toolchain received a handoff notice.

The VulnCheck VP said the quiet part: manual enrichment of new vulnerabilities is no longer feasible. The CISO from Contrast Security said the loud part: this is the end of the era where a single government database is the authoritative source of security risk. What neither said: every downstream system that assumed that era was still running is now operating on a formally-invalid premise.

Fitch says vulnerabilities will outnumber patches for the foreseeable future. The obvious response is speed: faster detection, faster coverage, faster patching. The less obvious response is semantic: audit your tooling for assumptions about the CVE database that are no longer true.

The map did not shrink. The legend changed. The organizations with the most sophisticated vulnerability management programs may be the ones most surprised — because they built the deepest dependency on a legend that was just quietly rewritten.
