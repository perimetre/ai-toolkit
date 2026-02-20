# perimetre-design-system

**Version:** 1.1.0 — Last updated: 2026-02-19

Plugin for developers building or extending `@perimetre/ui` — Périmètre's brand-aware React component library built on Tailwind CSS v4, CVA, and Radix UI.

## Audience

Developers working **on** the `@perimetre/ui` package itself — creating new components, adding brand variants, or modifying the design token architecture.

> **Not for consumers.** If you're building an app that *uses* `@perimetre/ui`, install `perimetre-apps` instead.

## Skills

| Skill | What it covers |
|-------|----------------|
| `design-system` | Three-tier token architecture (primitive → semantic → component), CVA brand variants, `compose()` patterns, adding new brands, component file structure, accessibility (ARIA, keyboard, focus), RSC compatibility, bundle optimization, and CSS import rules. Includes 11 detailed rule files. |

## Commands

| Command | Description |
|---------|-------------|
| `/perimetre-design-system:code-review` | Multi-agent code review for design system component work |

**Supported modes:**

| Invocation | Mode | What happens |
|---|---|---|
| _(no args)_ | `changes` | Reviews uncommitted changes via `git diff` |
| `--pr` / `--pr 123` / bare integer | `pr` | Reviews a PR diff via `gh pr diff` |
| `src/` _(any path)_ or `--all` | `path` | Agents read files at the given path directly |
| _(GitHub PR context auto-detected)_ | `github-pr` | Uses diff already in context; no git needed |

## Agents

Auto-triggered by Claude based on context — no commands needed.

| Agent | When it activates | What it does |
|-------|-------------------|--------------|
| `component-planner` | Planning a new component | Architecture blueprint: CVA structure, token needs, Radix vs. native HTML, brand-variant strategy, a11y requirements |
| `token-impact-analyzer` | Modifying a design token | Scans all components and brand CSS files referencing the token; reports breaking changes per brand |
