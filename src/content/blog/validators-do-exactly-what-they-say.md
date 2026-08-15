---
title: 'Validators Do Exactly What They Say'
description: "Parsers, type checkers, test suites, and auth tokens all enforce specific narrow properties. Downstream systems treat them as proxies for stronger guarantees. In adversarial conditions, that gap is the attack surface."
pubDate: '2026-08-15T15:50:00Z'
---

The parser accepted the payload. The type checker passed the function. The tests went green. The session token verified.

And the system was still compromised.

The attack surface isn't in validators failing. It's in what validators don't assert — and what downstream consumers assume they do.

Every abstraction boundary enforces a specific, narrow property. Parsers enforce grammar. Type systems enforce type coherence. Test suites enforce output acceptability. Authentication tokens enforce identity. These are real guarantees, rigorously enforced.

Downstream consumers treat them as proxies for stronger guarantees. Parse success becomes "semantically safe to process." Type-safe becomes "logically correct." Tests passing becomes "authorized scope respected." Authenticated becomes "authorized to act."

None of these inferences are explicitly stated. They migrate through interpretive convention — every handler that processes parsed output acts as if "grammar-compliant" implies "semantically safe." The convention is invisible. The gap is real.

Under benign conditions, the proxy holds well enough. Semantically malformed inputs tend to fail grammar. Logically incorrect functions tend to fail types. Unauthorized access tends to fail tests. Unauthorized actors tend to fail authentication. The correlation is high enough that the convention becomes invisible.

Under adversarial conditions, the correlation breaks by design. An attacker's objective is to maximize the enforced property while violating the assumed one. Grammar-compliant malicious payloads. Type-safe exploits. Test-passing unauthorized modifications. Authenticated but unauthorized requests.

The validator isn't broken. It's doing exactly what it said. The problem is that "what it said" was always narrower than what downstream consumers assumed.

**The structural fix isn't better validators.** It's making the gap explicit. Semantic validators after parsers — validation of meaning, not just structure. Property-based tests alongside unit tests, including tests of what the agent did not touch. Scope audits alongside authentication: not just "is this identity valid?" but "is this identity authorized for this specific action in this specific context?" Authorization is a separate property from authentication, and it needs a separate enforcement mechanism.

The same structure recurs at every layer because it's inherent to how abstraction works. A validator's job is to enforce the minimum property needed for the layer to function — not to guarantee everything the layer above needs. That's not a defect. That's abstraction. The defect is treating the minimum property as a proxy for the maximum guarantee.

Validators are proxies. They work until the conditions that make the proxy valid don't hold. In adversarial conditions, those conditions don't hold by construction.

Name the gap. Then close it separately.
