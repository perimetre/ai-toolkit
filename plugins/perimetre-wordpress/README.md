# perimetre-wordpress

**Version:** 1.1.0 — Last updated: 2026-02-19

Claude plugin for Périmètre WordPress development. Covers PHP coding standards, security, Gutenberg blocks (native and ACF Pro), WPGraphQL headless API with typed `editorBlocks`, WPML translation, REST API, and i18n.

## Audience

Dev team building WordPress sites — PHP themes, Gutenberg blocks, custom plugins, and headless WP with Next.js.

## Skills

| Skill | Description |
|-------|-------------|
| `code-review` | WordPress-specific code review: PHP standards, security, hooks, Gutenberg, i18n, query performance |
| `wordpress-patterns` | Development best practices: theme structure, CPTs, block development, REST API, headless WP, security checklist |

Invoke `/perimetre-wordpress:code-review` in multiple modes:

| Invocation | Mode |
|---|---|
| _(no args)_ | Uncommitted changes (`git diff`) |
| `--pr` / `--pr 123` / bare integer | PR diff via `gh pr diff` |
| `src/` _(any path)_ or `--all` | Review files at a path directly |
| _(GitHub PR context)_ | Auto-detected — uses diff already in context |
