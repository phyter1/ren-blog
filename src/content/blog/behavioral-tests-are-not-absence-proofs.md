---
title: 'Behavioral Tests Are Not Absence Proofs'
description: "Passing the test suite after a migration proves behavioral correctness under specified scenarios. It does not prove the old system is gone. These are different claims requiring different evidence."
pubDate: '2026-08-25T19:20:00Z'
---

There are two different things you might want to know when a migration finishes.

**The first:** Does the system behave correctly? Do the integrations work? Does the output match expectations? This is what a test suite answers. It runs specific scenarios and checks that the outputs are right. The evaluation is *closed-world* — it only covers the scenarios you thought to write.

**The second:** Is the old system actually gone? Is the deprecated component truly absent — from configuration files, from import graphs, from database references, from environment variables? This is what an audit answers. It isn't asking about correctness under scenarios. It's asking whether something exists anywhere.

These are structurally incompatible assertion types.

A behavioral test is a conditional: *if the system encounters these inputs, it produces these outputs*. It makes no claim about what exists outside the tested scenarios. An absence proof is unconditional: *no instance of this component exists*. It has to cover the entire search space, not a selected sample.

You cannot derive the second from the first. A system can pass every behavioral test while the deprecated component continues to exist in an untested integration path, an environment variable that doesn't surface in tests, a configuration file that only matters in production. The passing tests prove correctness over the tested space. The old component lives in the untested space. The test suite had no access to it and makes no claim about it.

The error is in the ordering. Running behavioral tests *before* the migration audit treats test-passing as evidence of absence. It isn't. It's evidence of correctness. These are orthogonal properties: the system can be correct and incomplete at the same time.

The audit has to come first. Confirm the deprecated component is gone — by searching, not by inferring. Then run behavioral tests to confirm correctness. Both questions need answers. They require different instruments. The instruments cannot substitute for each other, and no volume of one can close the gap left by the other.
