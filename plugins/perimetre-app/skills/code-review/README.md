# Code Review Skill — Périmètre App Plugin

The `code-review` skill provides multi-agent, confidence-scored code reviews for Périmètre Next.js/React projects.

## What It Does

Performs pragmatic code reviews for Next.js/React projects using Périmètre framework patterns:

1. **Examines changes** — reads `git status`, `git diff`, and recent commits
2. **Runs 3 parallel review agents** — each agent covers a different angle (pattern compliance, bugs, history)
3. **Scores confidence** — each issue gets a confidence score (0-100) via parallel Haiku agents
4. **Filters noise** — issues below confidence 80 are discarded
5. **Outputs structured results** — actionable issues with evidence, code diffs, and local references

## Files in This Skill

```
skills/code-review/
├── SKILL.md       ← Agent skill definition and step-by-step instructions
├── PATTERNS.md    ← Detailed pattern examples and anti-patterns for code review agents
├── README.md      ← This file
└── USAGE.md       ← Usage guide and configuration details
```

## How to Use

### As a slash command

```
/perimetre-code-review
```

This runs the full multi-agent review pipeline (3 agents + confidence scoring).

### As a skill (from other commands/skills)

```markdown
Use the `code-review` skill to review the current changes.
```

### Directly via SKILL.md

Agents can read `skills/code-review/SKILL.md` directly and follow its instructions.

## Agents

The skill orchestrates 3 defined agents from `agents/`:

| Agent | Model | Description |
|---|---|---|
| `pattern-reviewer` | Sonnet | Périmètre framework pattern compliance — all 11 pattern skills pre-injected at startup |
| `bug-scanner` | Haiku | Shallow bug scan — reads only the git diff, focuses on large obvious bugs |
| `history-reviewer` | Haiku | Historical context — reads git blame and history, identifies regressions |

## Patterns Covered

The `pattern-reviewer` agent checks for compliance with all 11 Périmètre pattern skills:

| Pattern Area | Checks |
|---|---|
| Error handling | Error-as-values, `ok: true as const`, no `ok: false` |
| Service layer | Business logic in services, thin routes, `defineService` |
| tRPC | Router patterns, middleware, prefetching |
| Forms | Single shared Zod schema, uncontrolled inputs, no inline validation |
| GraphQL | Factory pattern, codegen, cache invalidation |
| TanStack Query | `queryOptions` factory, server prefetch, `invalidateQueries` |
| Icons | `@perimetre/icons`, `currentColor`, accessibility at usage site |
| Images | `next/image`, `width`/`height`, `fill`/`sizes`, no SVG icons |
| SEO | `generateMetadata`, `metadataBase`, canonical URLs, JSON-LD |
| Caching | `"use cache"`, ISR, `revalidateTag`, no pure SSG |
| Payload CMS + Drizzle | Local API vs. direct Drizzle, typed schema |

## Sharing with Your Team

This skill is distributed as part of the `perimetre-app` Claude Code plugin. Install the plugin to get it:

1. Install the plugin in Claude Code's plugin marketplace or directly
2. All commands, skills, and agents become available automatically

See `README.md` at the plugin root for installation instructions.
