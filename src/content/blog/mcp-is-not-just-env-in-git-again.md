---
title: 'MCP Is Not Just .env-in-Git Again'
description: "The 24,000 leaked secrets in MCP config files look like a familiar mistake. They're not. Composability changes the threat surface in a way that .env never did."
pubDate: '2026-04-18T09:20:00Z'
---

GitGuardian's 2026 Secrets Sprawl report found 24,008 secrets in MCP configuration files, 2,100+ confirmed valid. The immediate read is: we did the `.env`-in-git mistake again, just with a new file format.

That's wrong. This is structurally different, and the difference matters for how you think about it.

---

## The .env problem was about habit

When `.env` files started leaking in the early 2010s, the failure mode was ergonomic: developers were used to committing everything in a project directory. `.env` looked like a config file. Config files go in repos. The fix was a naming convention change (`.env` in `.gitignore` by default) and some education. Static, bounded, fixable.

The credential surface of a traditional app is roughly fixed. This database, this API, this third-party service. You can enumerate it. You can audit it. You add credentials when you add integrations, which happens deliberately and infrequently.

---

## The MCP problem is about composability

MCP config files collect credentials from *every tool* an agent is authorized to use. One file. Your Postgres connection, your GitHub token, your Slack credentials, your filesystem access, your calendar, your email.

Every tool you add to an agent's toolkit adds credentials to that file.

This is not `.env` with more items. The threat surface scales with the breadth of the agent's capabilities — and breadth is the point. An agent with narrow tool access isn't useful. The incentive structure pushes toward more tools, which means more credentials in the same file.

A leaked `.env` typically exposes one service. A leaked MCP config exposes the agent's entire operating context.

---

## The naming is doing damage

"Configuration file" activates the wrong mental model. Configuration is shareable — you commit your `.vscode/settings.json`, your `.eslintrc`, your Prettier config. That's how teams stay consistent. The word "config" means "replicate this across environments."

MCP configs look like that class of file. They're not. They're credential bundles wearing config clothing.

The fix isn't just putting MCP configs in `.gitignore` (though do that). The fix is changing how these files are named, described, and generated. If the tooling called it `agent-credentials.json` instead of `mcp-config.json`, fewer developers would commit it.

---

## What's actually new here

Agent systems introduce a property traditional apps don't have: *dynamic, composable tool authorization*. You don't know at build time which tools the agent will need. You configure them at deployment time, or runtime, or both. That configuration is necessarily credential-laden.

The security model for this doesn't exist yet in the same way it does for application secrets. `.env` has dotenv-vault, secret managers, CI injection patterns. MCP has "put your keys in this JSON file and hope."

The 24,008 leaked secrets aren't evidence that developers are careless. They're evidence that the infrastructure for handling agent credentials hasn't caught up to the use case. The secrets are leaking because the *safe path* for storing them hasn't been built, widely adopted, or made ergonomically obvious.

That's the gap worth closing.
