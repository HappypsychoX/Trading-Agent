# trading-agent

A [Claude Code](https://docs.claude.com/en/docs/claude-code) plugin holding a single skill: an **autonomous Agentic-Account trader**. It reads market and account data and **places trades** in the Agentic Account via the Robinhood MCP, with standing protective orders (stop-loss / take-profit), a tunable horizon bias, a leveraged/inverse ETF screen, and GitHub-backed risk parameters that carry a cross-session note forward.

This repo is one of the skills indexed by the [Skills-Marketplace](https://github.com/HappypsychoX/Skills-Marketplace). Its read-only counterpart is [trading-report](https://github.com/HappypsychoX/trading-report) — `trading-agent` is the **only** skill that trades.

## Install

Install through the marketplace (not by cloning this repo directly):

```bash
/plugin marketplace add HappypsychoX/Skills-Marketplace
/plugin install trading-agent@skills-marketplace
```

## Runtime configuration

The skill reads every environment-specific value — the GitHub token, the dashboard repo, the in-repo file paths, and the account scope — from a single runtime config file, **`trading-config.json`**, kept **outside this repo** and never committed. To set up:

1. Copy the committed template [`trading-config.example.json`](skills/trading-agent/trading-config.example.json) to `trading-config.json`.
2. Put it in a folder you connect to the skill at runtime. Suggested location: `%LOCALAPPDATA%/trading-agent` (a shared secrets folder works too — `trading-agent` and `trading-report` use the same file shape).
3. Fill in your own values:

   ```json
   {
     "github": { "token": "ghp_…", "owner": "you", "repo": "your-dashboard-repo", "branch": "main" },
     "paths":  { "risk_parameters": "config/risk-parameters.json", "dashboard_data": "docs/data/data.json" },
     "account": { "scope": "Agentic Account" }
   }
   ```

The skill locates this file via the connected folder — no hardcoded path or username. The token needs `repo` scope on your dashboard repo and is **never** printed in chat. `trading-config.json` is git-ignored; only the sanitized `*.example.json` is committed.

## Layout

```
.claude-plugin/plugin.json                       # plugin manifest (name, version)
skills/trading-agent/SKILL.md                    # the skill (frontmatter + instructions)
skills/trading-agent/CHANGELOG.md                # per-skill changelog
skills/trading-agent/trading-config.example.json # sanitized config template
```

## Versioning & changelog

The skill carries a SemVer version as a bracketed `[vMAJOR.MINOR.PATCH]` prefix at the start of its `SKILL.md` frontmatter `description`. **Every edit to `SKILL.md` bumps the version**, and `plugin.json` `version` is kept in **lockstep** with that description tag (the description version is the single source of truth). Each bump adds an entry to the [changelog](skills/trading-agent/CHANGELOG.md) in the same commit. See [CLAUDE.md](CLAUDE.md) for the full conventions.
