# perimetre-wordpress

**Version:** 1.2.0 — Last updated: 2026-02-19

Plugin for WordPress development at Périmètre. Covers PHP coding standards, security, Gutenberg blocks (native and ACF Pro), WPGraphQL, WPML, REST API extensions, and i18n.

## Audience

Developers building WordPress sites — PHP themes, Gutenberg blocks, custom plugins, and headless WordPress with Next.js.

> For headless projects where all code is on the JS side (consuming a WP REST/GraphQL API), install `perimetre-apps` instead.

## Skills

| Skill | What it covers |
|-------|----------------|
| `code-review` | WordPress-specific code review: PHP coding standards (WPCS), security (nonces, capability checks, sanitization, escaping), hook patterns, Gutenberg blocks, REST API, i18n compliance, and query performance |
| `wordpress-patterns` | Development best practices: theme structure, custom post types and taxonomies, Gutenberg block development with `@wordpress/scripts`, ACF Pro, WPGraphQL typed `editorBlocks`, WPML translation strategies, REST API extensions, headless WP + Next.js integration, and security checklist |

## Commands

| Command | Description |
|---------|-------------|
| `/perimetre-wordpress:code-review` | Multi-agent WordPress code review across 5 specialized agents |

**Supported modes:**

| Invocation | Mode | What happens |
|---|---|---|
| _(no args)_ | `changes` | Reviews uncommitted changes via `git diff` |
| `--pr` / `--pr 123` / bare integer | `pr` | Reviews a PR diff via `gh pr diff` |
| `src/` _(any path)_ or `--all` | `path` | Agents read files at the given path directly |
| _(GitHub PR context auto-detected)_ | `github-pr` | Uses diff already in context; no git needed |

**Review agents (run in parallel):**

| Agent | What it checks |
|-------|----------------|
| `security-reviewer` | Nonces, capability checks, sanitization, escaping, REST API permissions, DB query safety |
| `wpcs-reviewer` | PHP naming conventions, Yoda conditions, type declarations, tabs, DocBlocks |
| `gutenberg-reviewer` | `block.json`, dynamic blocks, `useBlockProps`, `InspectorControls`, `@wordpress/scripts` |
| `hooks-history-reviewer` | Lifecycle hook usage, prefixed callbacks, and git-based regression detection |
| `i18n-performance-reviewer` | Text domain compliance, string wrapping, and `WP_Query` efficiency |

## Agents

Auto-triggered by Claude based on context — no commands needed.

| Agent | When it activates | What it does |
|-------|-------------------|--------------|
| `block-planner` | Planning a new Gutenberg block | Architecture blueprint: native vs. ACF decision, field structure, WPGraphQL schema, WPML strategy, rendering approach, and `block.json` skeleton |
| `cpt-taxonomy-planner` | Planning custom post types or taxonomies | CPT registration args, taxonomy structure, REST API visibility, WPGraphQL type names, WPML strategy, and required hooks |
