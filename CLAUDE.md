# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A single **Claude Code plugin** holding one skill, `trading-agent`, indexed by the [Skills-Marketplace](https://github.com/HappypsychoX/Skills-Marketplace) (which references this repo via a `github` plugin `source`). There is no application, build system, or test suite — the deliverable is the skill itself.

The skill lives as a plain `SKILL.md`: Markdown with a YAML frontmatter block (`name`, `description`) followed by the instruction body. The `description` is what a Claude agent matches against to decide when to invoke the skill, so it is written as a long trigger-phrase list, not prose.

```
.claude-plugin/plugin.json                       # plugin manifest (name, version)
skills/trading-agent/SKILL.md                    # the skill — edit here
skills/trading-agent/CHANGELOG.md                # per-skill changelog
skills/trading-agent/trading-config.example.json # sanitized config template
```

The skill under `skills/` is auto-discovered — it is not listed in `plugin.json`.

## What the skill does

`trading-agent` is the **autonomous trading agent**. It reads market/account data and **places trades** in the Agentic Account via the Robinhood MCP. It fetches its risk parameters fresh each session from the configured `paths.risk_parameters` file (e.g. `config/risk-parameters.json`), falling back to a hardcoded table.

This is the **only** skill that trades. Its read-only counterpart, [trading-report](https://github.com/HappypsychoX/trading-report), must never place or mutate orders — keep that read/write boundary intact if a change touches shared conventions (repo names, file paths, config shape), and mirror such changes across both repos.

Infrastructure the skill assumes:
- **Robinhood MCP** for account/market data and order placement.
- **GitHub REST Contents API** — reads/writes go to the dashboard repo configured in `trading-config.json` (owner/repo/branch), **not** to this repo. The skill uses the raw REST API (`curl`), never `git` clones or local checkouts, so it runs identically from a fresh unattended session.
- **Scope: the configured account only** (`account.scope`, default "Agentic Account"). Other Robinhood accounts must never appear in queries or output.

## Runtime configuration

One external runtime config file, **`trading-config.json`**, holds every environment-specific value — the GitHub token (`github.token`, shape `{"github": {"token": "ghp_..."}}` plus `owner`/`repo`/`branch`), the in-repo file paths (`paths.risk_parameters`, `paths.dashboard_data`), and the account scope (`account.scope`). It lives **outside this repo**, located at runtime via a connected folder (suggested `%LOCALAPPDATA%/trading-agent`) — never a hardcoded path or username. Only the sanitized `trading-config.example.json` template is committed; the real file is git-ignored. **Never print the token in chat.**

**Never hardcode a machine-specific filesystem path or username** in the skill body — locate `trading-config.json` (and any notes folder) via whatever folder is connected, so the same skill text runs across different PCs.

## Conventions to preserve when editing the skill

### Versioning

The skill carries a SemVer version as a bracketed prefix at the very start of its `SKILL.md` frontmatter `description`:

```
[vMAJOR.MINOR.PATCH] <existing trigger-phrase description…>
```

**Every edit to `SKILL.md` bumps the version — no silent changes.** Bump by impact:

| Part | Bump when… |
| --- | --- |
| **MAJOR** | A breaking behavior change — the read-only vs. write boundary moves, a capability is removed/renamed, or required infra (repo, paths, token shape) changes. |
| **MINOR** | A backward-compatible capability or new trigger phrases are added. |
| **PATCH** | Wording, clarifications, or instruction fixes with no behavioral change. |

**Keep `plugin.json` `version` in lockstep with the description tag**, using the description version as the single source of truth: on every skill edit, set both the `SKILL.md` `[vX.Y.Z]` and `plugin.json` `version` to the same bumped SemVer. This repo is one skill per plugin, so one honest number is clearer than two, the `/plugin` picker shows the version the skill advertises, and — because a static unchanged `plugin.json` `version` means installers don't receive the update — tying the plugin bump to the mandatory description bump guarantees a skill change can't ship without moving the version that gates delivery.

### Changelog

`skills/trading-agent/CHANGELOG.md` follows [Keep a Changelog](https://keepachangelog.com/) style, newest first. **Every version bump adds an entry in the same commit as the `SKILL.md`/`plugin.json` change** — no version moves without one:

```markdown
## [X.Y.Z] — YYYY-MM-DD
- **Changed/Added/Removed/Fixed:** <one line per change>
```

- The heading version must equal the new `[vX.Y.Z]` description tag and `plugin.json` `version` — all three move together.
- MAJOR changes go under **Changed**/**Removed**, MINOR under **Added**/**Changed**, PATCH under **Fixed**/**Changed**.
- Keep entries user-facing: describe what the skill now does differently, not the wording diff.

### Triggering surface

The `description` frontmatter is a triggering surface: when adding capabilities, **extend** its trigger phrases rather than shortening it.
