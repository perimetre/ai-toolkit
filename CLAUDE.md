# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

A **git-based Claude plugin marketplace** for Périmètre. It distributes specialized skills, agents, and commands to Claude users via the Claude plugin system. There is no build step — committing to `main` is the deployment.

## Plugin Marketplace Architecture

The root `.claude-plugin/marketplace.json` is the central registry listing all plugins. Each plugin lives under `plugins/[plugin-name]/` with its own `.claude-plugin/plugin.json`.

**Plugin anatomy:**
```
plugins/[plugin-name]/
├── .claude-plugin/plugin.json    # Plugin metadata (name, version, description)
├── README.md                      # Plugin documentation
├── skills/[skill-name]/
│   ├── SKILL.md                  # Skill definition — YAML frontmatter + rich content
│   └── rules/                    # Optional supplementary rule files
├── commands/[command].md         # Slash command definitions
└── agents/[agent].md             # Multi-agent configurations
```

**SKILL.md format:** YAML frontmatter with `name`, `description`, `tools`, `skills` fields, followed by the skill body Claude reads as context.

## Installed Plugins

| Plugin | Purpose | Key Skills |
|--------|---------|------------|
| `perimetre-apps` | Full-stack JS/TS/React/Next.js | error-handling (discriminated unions), services, tRPC, forms, TanStack Query, Vercel rules |
| `perimetre-brand-guidelines` | Visual identity guidelines | brand-guidelines (colours: Carbon #111111, Aqua #13FADC; Satoshi typeface) |
| `perimetre-design-system` | `@perimetre/ui` component library development | CVA patterns, three-tier token system, multi-brand CSS with `data-pui-brand` |
| `perimetre-wordpress` | WordPress/PHP development | code-review, wordpress-patterns |
| `perimetre-documents` | Professional document writing | document (pairs with perimetre-brand-guidelines) |

## Multi-Agent Code Review Pattern

The code review commands (`/perimetre-apps:code-review`, `/perimetre-design-system:code-review`, `/perimetre-wordpress:code-review`) support four modes:

| Invocation | Mode | Behavior |
|---|---|---|
| _(no args)_ | **changes** | Reviews current uncommitted changes via `git diff` |
| `--pr` / `--pr 123` / bare integer | **pr** | Reviews a PR diff via `gh pr diff` (never `git diff main...HEAD`) |
| `src/` _(any path)_ or `--all` | **path** | Agents read files directly; history agents skipped |
| _(GitHub PR context auto-detected)_ | **github-pr** | Uses diff already in context; skips Step 1 and history agents |

**Architecture:**

1. **Step 0** — Detects mode (GitHub context → explicit args → default `changes`)
2. **Step 1** — A Haiku agent gathers scope (skipped in `github-pr` mode)
3. **Step 2** — 2–5 specialized agents run **in parallel**; history agents are skipped in `github-pr` and `path` modes
4. **Step 3** — Each finding gets a confidence score (0–100)
5. **Step 4** — Issues below 80 confidence are filtered out
6. **Step 5** — Output is formatted with evidence, doc citations, and a **Mode** line

Agent definitions live in `plugins/[plugin]/agents/[agent].md`. They use constrained tools (Read, Grep, Glob, Bash) and inject skill documentation as context.

## Working with Plugins

When creating, reviewing, or refactoring any plugin component (plugin.json, SKILL.md, commands, agents, rules), always consult the official Claude Plugin documentation first: https://code.claude.com/docs/en/plugins-reference

## CRITICAL: After Every Plugin Change

**NEVER forget these steps when modifying any plugin:**

1. **Bump the version number** in `plugins/[plugin-name]/.claude-plugin/plugin.json`
2. **Update the plugin's README.md** to reflect changes
3. **Update the root marketplace README.md** if the change is significant
4. **READMEs must always show the current version number and last update date**

These steps are mandatory — skipping them leaves the marketplace in an inconsistent state.

## Adding a New Plugin

1. Create `plugins/[plugin-name]/` with the directory structure above
2. Add an entry to `.claude-plugin/marketplace.json`
3. Commit to `main` — the plugin is immediately available

## Key Files

- `.claude-plugin/marketplace.json` — Central plugin registry
- `plugins/perimetre-apps/skills/error-handling/SKILL.md` — 950+ line error-handling reference (Go/Rust-style discriminated unions)
- `plugins/perimetre-design-system/skills/design-system/rules/` — 11 rule files for the token architecture and brand system
