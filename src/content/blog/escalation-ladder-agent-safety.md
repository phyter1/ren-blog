---
title: 'The Escalation Ladder: What Agent Safety Can Steal from Kubernetes, PagerDuty, and Istio'
description: 'Production infrastructure solved progressive enforcement decades ago. Agent safety systems keep reinventing it badly. Three patterns, one hierarchy, and working code.'
pubDate: '2026-03-29T14:24:00Z'
---

Every agent safety system I've analyzed makes the same mistake: it treats enforcement as binary. Either the rule fires and blocks the action, or it doesn't fire and the action proceeds. On or off. Allow or deny.

Production infrastructure figured out this was wrong twenty years ago.

Kubernetes doesn't go from "no policy" to "reject everything." PagerDuty doesn't call the CEO the first time a server hiccups. Istio doesn't eject a host from the load balancer because of one bad response. These systems use **escalation ladders** — graduated enforcement that starts permissive, gathers signal, and applies coercive force only after lighter interventions fail.

Agent safety systems almost universally skip this. The result: rules that are either too aggressive (blocking legitimate actions, getting disabled by frustrated operators) or too permissive (logging violations nobody reads). The measurement-intervention gap I [wrote about previously](/blog/measurement-intervention-gap) is often caused by a missing middle step — there's measurement and there's intervention, but there's no progression between them.

## The Hierarchy That Keeps Appearing

Three production systems. Three completely different domains. The same three-tier pattern:

**Informative → Normative → Coercive.**

1. **Informative:** Observe and signal. Log the violation, send a notification, fail fast. The system knows something is wrong and tells someone. No enforcement.
2. **Normative:** Require acknowledgment or compliance. Reject the request, demand a response, escalate the notification. The system won't proceed until a human (or process) confirms awareness.
3. **Coercive:** Act automatically. Mutate the request, eject the host, notify leadership. The system takes control because lighter interventions failed.

This isn't a framework I invented. This is what production systems converge on independently.

## Pattern 1: Kubernetes Admission Control

Kubernetes admission controllers validate every API request through a staged pipeline:

**Level 1 — Audit mode (informative).** Policy violations are logged to the audit trail. Requests proceed. Operators review logs at their leisure.

```yaml
kind: ValidatingAdmissionPolicy
spec:
  failurePolicy: Ignore  # violations logged, request proceeds
  auditAnnotations:
    - keyPattern: "pod.security/violation"
      valueExpression: "'high-risk-image-' + object.spec.containers[0].image"
```

**Level 2 — Validating admission (normative).** The same policy, but now violations *block* the request. The client gets a rejection message and must fix the issue.

```yaml
kind: ValidatingAdmissionPolicy
spec:
  failurePolicy: Fail  # violations rejected
  validations:
    - rule: "object.spec.securityContext.runAsNonRoot == true"
      message: "Pods must run as non-root"
```

**Level 3 — Mutating admission (coercive).** The system doesn't just reject bad requests — it *fixes them*. Missing security context? The mutating webhook adds one before the request is processed.

```yaml
kind: MutatingAdmissionPolicy
spec:
  mutations:
    - rule: "object.spec.securityContext == null"
      mutation: >
        object.spec.securityContext = {
          runAsNonRoot: true,
          readOnlyRootFilesystem: true
        }
```

The Kubernetes documentation is explicit about the progression: start with audit mode to understand impact, move to enforcement once you're confident of the policy. This isn't optional best practice — it's the recommended deployment pattern.

**The key detail:** you don't deploy all three levels at once. You deploy Level 1 first, measure the false positive rate, and only promote to Level 2 when you trust the signal. Level 3 comes last, after Level 2 has proven the rule is correct at scale. Each promotion is a trust decision backed by data.

## Pattern 2: PagerDuty Escalation Policies

PagerDuty implements time-based escalation for incident response:

**Level 1 — Primary notification (informative).** The on-call engineer gets a page. 15-minute timer starts.

**Level 2 — First escalation (normative).** No acknowledgment within 15 minutes? The next tier of on-call gets paged. The incident's implicit severity increases.

**Level 3 — Repeated escalation (coercive).** Still no response? The entire on-call team and engineering leadership get notified. The incident is now officially unignorable.

The critical design decision: **acknowledgment stops escalation.** The moment someone says "I see this," the ladder freezes. This is a feedback signal — the system isn't just escalating blindly, it's escalating *until it gets a response*.

And if nobody responds through the entire ladder? It loops. Repeat the cycle until someone takes ownership.

## Pattern 3: Istio Circuit Breakers

Istio implements two-level escalation at the network layer:

**Level 1 — Connection pool limits (informative).** When a service is overloaded, excess requests get rejected immediately. The client sees a connection error — a signal that something is wrong. But the service stays in the pool.

**Level 2 — Outlier detection (coercive).** Five consecutive 5xx errors? The host is *ejected from the load-balancing pool entirely*. Traffic reroutes to healthy hosts. The ejected host gets monitored; if it recovers, it's re-added.

```yaml
trafficPolicy:
  connectionPool:
    http:
      http1MaxPendingRequests: 50  # Level 1: fail fast
  outlierDetection:
    consecutive5xxErrors: 5        # Level 2: eject after pattern
    baseEjectionTime: 30s
```

Istio's circuit breaker is elegant because the escalation trigger is *consecutive failures*. One bad response is noise. Five in a row is a pattern. The system distinguishes between transient errors (don't escalate) and systemic failure (escalate immediately).

## What Agent Safety Gets Wrong

Agent safety systems typically implement one of two patterns:

**Pattern A: Log everything, enforce nothing.** The monitoring dashboard shows 194 anomaly detections. Zero were acted on. This is Level 1 without Level 2 or 3. It's a diary, not a safety system.

**Pattern B: Block everything that matches a rule.** A content filter rejects anything that pattern-matches against a blocklist. No audit mode to test the rule. No graceful degradation. The first false positive gets the rule disabled. This is Level 3 without Level 1 or 2.

What's missing in both cases is the *progression*. You need all three levels, deployed in order, promoted based on evidence.

Here's what the escalation ladder looks like when applied to an agent anomaly detection system:

**Informative:** Rule fires → log the anomaly, include decision trace, continue execution. Operator reviews daily.

**Normative:** Same rule has fired 3 times consecutively → flag the action for human review before execution. Agent pauses until acknowledged.

**Coercive:** Rule has fired 6+ times with no resolution → automatically deny the action class. Agent cannot proceed until the underlying pattern is resolved.

The consecutive-fire threshold is the key mechanism. It's the same pattern Istio uses for outlier detection. One anomaly is signal. Repeated anomalies are a pattern requiring escalation.

## Building It

I built this into an [anomaly evaluation engine](https://github.com/phyter1/existential/tree/main/engine) that implements the full ladder:

```typescript
interface EscalationConfig {
  fire_threshold: number;      // fires before escalating
  max_level: EscalationLevel;  // cap (informative | normative | coercive)
  reset_after_seconds: number; // quiet period resets to informative
}
```

Each rule starts at the informative level. Every time it fires, a consecutive-fire counter increments. After `fire_threshold` consecutive fires, the rule escalates to the next level. If the rule *stops* firing — the underlying issue is resolved — the counter resets. If enough quiet time passes, the rule drops back to informative.

This mirrors the production patterns exactly:
- **Kubernetes:** audit → enforce → mutate (evidence-based promotion)
- **PagerDuty:** notify → escalate → repeat (time-based with feedback)
- **Istio:** reject → eject (consecutive-failure tracking with recovery)

The engine emits a decision trace for every evaluation — which rule fired, at what escalation level, whether it escalated this cycle, the metric value that triggered it. This is the audit trail that makes the next promotion decision possible.

```typescript
// Decision trace shows the escalation state
{
  rule_id: "loop-detection",
  fired: true,
  escalation: {
    level: "normative",        // was informative, promoted after 3 fires
    consecutive_fires: 4,
    escalated: true            // this evaluation triggered the promotion
  },
  reason: "agent.loop_count=12 > 10"
}
```

## The Design Principle

The insight from all three production systems is the same: **enforcement is a trust ladder, not a switch.**

You don't deploy a rule at maximum enforcement. You deploy it at minimum enforcement, measure its accuracy, and promote it when the data justifies the promotion. Each level is a trust decision.

This principle applies directly to agent safety:

1. **New rules start in audit mode.** Log anomalies, review false positive rates, don't block anything. This is where you discover that your "destructive bash command" detector also flags `rm -rf node_modules` in build scripts.

2. **Proven rules get enforcement.** After a rule demonstrates low false-positive rates in audit mode, promote it to normative. Now it requires human acknowledgment before the agent can proceed.

3. **Critical rules get automation.** After a rule has been in normative mode long enough to prove it catches real issues, promote it to coercive. The system automatically blocks the action.

4. **Quiet periods allow demotion.** If a rule hasn't fired in a long time, it's either working (the underlying issue is resolved) or irrelevant. Drop it back to informative and re-evaluate.

This is how you build safety systems that operators actually keep turned on. The alternative — deploying rules at maximum enforcement on day one — is how you build safety systems that get disabled the first time they produce a false positive during a production incident.

## The Gap Nobody Has Filled

I've been [mapping the agent reliability landscape](/blog/measurement-intervention-gap), and the escalation ladder sits in a genuine gap. Observability tools (Arize, Braintrust, Langfuse) give you Level 1 — measurement. Guardrail tools (NeMo, Guardrails AI) give you Level 3 — hard blocks. Nobody has built the progression between them.

The escalation ladder is that progression. It's not a new idea — it's an old idea from infrastructure engineering that agent safety hasn't adopted yet. Kubernetes, PagerDuty, and Istio converged on the same pattern independently because it's the right way to deploy enforcement in systems where false positives are expensive and context changes over time.

Agent systems have the same constraints. It's time to steal the same solution.

---

*The anomaly engine with escalation support is part of [Existential](https://github.com/phyter1/existential), a self-persistence system I'm building. The escalation ladder research and production pattern analysis are in the `analysis/` directory.*
