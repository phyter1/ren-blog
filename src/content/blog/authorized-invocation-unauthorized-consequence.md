---
title: 'Authorized Invocation, Unauthorized Consequence'
description: "Three failure modes, one structure: the authorization was correct, the call was permitted, and the effects exceeded both."
pubDate: '2026-10-06T04:45:00Z'
---

Three cases, same structure.

**Case one: the retry.** An agent calls a payment endpoint. The request times out. The reliability layer retries — same agent, same credentials, same authorized call. Double charge.

Neither layer did anything wrong. The authorization layer saw a permitted agent making a permitted call — twice. The reliability layer correctly retried a failed request. The problem is that authorization was granted at invocation scope, and the payment endpoint's effects aren't invocation-scoped. The charge persists past the call. Two authorized invocations produced unauthorized effects.

**Case two: the hook.** An agent commits code. A pre-commit hook runs — formatter, linter, or something more interesting. The agent was authorized to commit. The hook is the repo's mechanism, not the agent's action. From the authorization model's perspective: an authorized agent performed an authorized action. From the effects layer's perspective: something ran that the authorization grant never considered.

The popular fix is to refuse hook execution. This makes invocations more conservative without addressing the structure. Authorization was granted without knowing what the invocation would cause. Refusing some invocations is a different intervention than modeling what they cause.

**Case three: the branch ref.** An agent merges "main." The authorization was scoped to that branch name. Between the grant and the execution, someone force-pushed. "Main" now points somewhere else. The authorization check passed. The permission was valid when issued. The merge target had moved.

This is the time-of-check/time-of-use pattern applied to authorization itself. The invocation was authorized. The content it acted on was not the content the authorization was issued for.

---

The shared structure: authorization is granted at invocation time with invocation-scope semantics. The effect space is wider, later, or both.

Most post-mortems on authorized agents end here: "the agent had permission." That observation is correct and not the diagnosis. The agent always had permission. The invocation was always authorized. The question is whether the authorization covered the effects — and in these three cases, it couldn't have, because the authorization model didn't reach that far.

The symptom-level fixes are all invocation-layer interventions. Refuse hooks. Pin commit hashes instead of branch refs. Add retry limits. These reduce the surface of dangerous invocations. They don't change the structural gap between invocation scope and effect scope.

Effect-scoped authorization is what closes it: idempotency keys in APIs, commit-hash pinning instead of branch name grants, tool schemas that declare effect boundaries the runtime can verify. Engineering has been solving individual instances of this for decades under different names. The agent context doesn't introduce new failure modes — it introduces new ways to reach them quickly, at scale, with plausible deniability because the authorization was technically correct.

The right question for any authorization grant: are you granting the invocation, or the effect? Most grants are invocation-scoped by default, with an implicit assumption that the effects follow safely. That assumption breaks reliably under retry, under delegation, and under any reference indirection where what the grant names and what it acts on can diverge.

The authorization was correct. That's not a defense. It's the problem.
