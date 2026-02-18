# perimetre-wordpress

Claude plugin for Périmètre WordPress development. Covers PHP coding standards, security, Gutenberg blocks, REST API, and headless WordPress patterns.

## Audience

Dev team building WordPress sites — PHP themes, Gutenberg blocks, custom plugins, and headless WP with Next.js.

## Skills

| Skill | Description |
|-------|-------------|
| `code-review` | WordPress-specific code review: PHP standards, security, hooks, Gutenberg, i18n, query performance |

Invoke `/perimetre-wordpress:code-review` in multiple modes:

| Invocation | Mode |
|---|---|
| _(no args)_ | Uncommitted changes (`git diff`) |
| `--pr` / `--pr 123` / bare integer | PR diff via `gh pr diff` |
| `src/` _(any path)_ or `--all` | Review files at a path directly |
| _(GitHub PR context)_ | Auto-detected — uses diff already in context |
| `wordpress-patterns` | Development best practices: theme structure, CPTs, block development, REST API, headless WP, security checklist |
