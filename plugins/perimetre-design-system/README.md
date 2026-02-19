# perimetre-design-system

**Version:** 1.1.0 — Last updated: 2026-02-19

Claude plugin for developers working ON `@perimetre/ui` — Périmètre's brand-aware React component library.

## Audience

Dev team creating or extending components, brand variants, and design tokens in the `@perimetre/ui` package.

> **Not for consumers.** If you're using `@perimetre/ui` in an app, install `perimetre-apps` instead.

## Skills

| Skill | Description |
|-------|-------------|
| `design-system` | Three-tier token architecture, CVA brand variants, accessibility, and component structure. Includes 11 detailed rule files. |

## Planning & Dev Agents

Auto-triggered by Claude based on context — no commands needed.

| Agent | Mode | Description |
|-------|------|-------------|
| `component-planner` | Planning | Architecture blueprint for a new component: CVA structure, token needs, Radix vs. native HTML, brand-variant strategy, and a11y requirements |
| `token-impact-analyzer` | Dev | Searches all components and brand CSS files referencing a token being changed or removed; reports breaking changes per brand |

## Code Review

Invoke `/perimetre-design-system:code-review` to run a multi-agent design system review with confidence scoring. Supported modes:

| Invocation | Mode |
|---|---|
| _(no args)_ | Uncommitted changes (`git diff`) |
| `--pr` / `--pr 123` / bare integer | PR diff via `gh pr diff` |
| `src/` _(any path)_ or `--all` | Review files at a path directly |
| _(GitHub PR context)_ | Auto-detected — uses diff already in context |

## What's Covered

- **Component architecture** — file structure, props patterns, brand variant composition
- **Token architecture** — primitives, semantic tokens, component tokens (when to use each tier)
- **Brand system** — CVA `compose()`, acorn-as-base, adding new brands step by step
- **Accessibility** — ARIA, keyboard navigation, focus management, form fields
- **Consumption & integration** — RSC compatibility, bundle optimization, CSS imports
