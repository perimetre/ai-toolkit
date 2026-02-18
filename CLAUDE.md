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
| `perimetre-app` | Full-stack JS/TS/React/Next.js | error-handling (discriminated unions), services, tRPC, forms, TanStack Query, Vercel rules |
| `perimetre-brand` | Visual identity guidelines | brand-guidelines (colours: Carbon #111111, Aqua #13FADC; Satoshi typeface) |
| `perimetre-design-system` | `@perimetre/ui` component library development | CVA patterns, three-tier token system, multi-brand CSS with `data-pui-brand` |
| `perimetre-wordpress` | WordPress/PHP development | code-review, wordpress-patterns |
| `perimetre-document` | Professional document writing | document (pairs with perimetre-brand) |

## Multi-Agent Code Review Pattern

The code review commands (`/perimetre-app:code-review`, `/perimetre-design-system:review`, `/perimetre-wordpress:code-review`) follow this architecture:

1. An orchestrator examines the git diff
2. 2–5 specialized agents run **in parallel** (pattern-reviewer, bug-scanner, security-reviewer, etc.)
3. Each finding gets a confidence score (0–100)
4. Issues below 80 confidence are filtered out
5. Output is formatted with evidence and doc citations

Agent definitions live in `plugins/[plugin]/agents/[agent].md`. They use constrained tools (Read, Grep, Glob, Bash) and inject skill documentation as context.

## Working with Plugins

When creating, reviewing, or refactoring any plugin component (plugin.json, SKILL.md, commands, agents, rules), always consult the official Claude Plugin documentation first: https://claude.ai/plugins/docs

## Adding a New Plugin

1. Create `plugins/[plugin-name]/` with the directory structure above
2. Add an entry to `.claude-plugin/marketplace.json`
3. Commit to `main` — the plugin is immediately available

## Key Files

- `.claude-plugin/marketplace.json` — Central plugin registry
- `plugins/perimetre-app/skills/error-handling/SKILL.md` — 950+ line error-handling reference (Go/Rust-style discriminated unions)
- `plugins/perimetre-design-system/skills/design-system/rules/` — 11 rule files for the token architecture and brand system
