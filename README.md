# AI Toolkit

**Version:** 0.1.0 — Last updated: 2026-02-19

Périmètre's internal plugin marketplace for Claude.

## Installation

Add this marketplace to Claude:

```
/plugin marketplace add https://github.com/perimetre/ai-toolkit
```

## Plugins

### `perimetre-apps`

For the dev team building apps and prototypes with the Périmètre JS/TS/React/Next.js stack.

**Skills:** `error-handling`, `services`, `trpc`, `forms`, `graphql`, `tanstack-query`, `icons`, `images`, `seo`, `caching`, `payload-drizzle`, `code-review`, `vercel-react-best-practices`, `vercel-composition-patterns`, `web-design-guidelines`, `web-animation-design`, `emil-design-engineering`

**Commands:** `/perimetre-apps:code-review`

**Agents:** `feature-planner` (planning), `scaffold-advisor` (dev)

### `perimetre-brand-guidelines`

Loads Périmètre's visual identity guidelines into context — colours, typography, and logo usage — so every deliverable is on-brand.

**Skills:** `brand-guidelines`

### `perimetre-design-system`

For the dev team working on `@perimetre/ui` — Périmètre's brand-aware React component library. Not for consumers; use `perimetre-apps` if you're building apps that use the design system.

**Skills:** `design-system`

**Commands:** `/perimetre-design-system:code-review`

**Agents:** `component-planner` (planning), `token-impact-analyzer` (dev)

### `perimetre-documents`

For anyone writing professional documents — reports, proposals, and presentations. Brand-agnostic by design; pair with `perimetre-brand-guidelines` for on-brand output.

**Skills:** `document`

### `perimetre-wordpress`

For the dev team building WordPress sites — PHP themes, Gutenberg blocks, custom plugins, and headless WP with Next.js.

**Skills:** `code-review`, `wordpress-patterns`

**Commands:** `/perimetre-wordpress:code-review`

**Agents:** `block-planner` (planning), `cpt-taxonomy-planner` (planning)

### `perimetre-discover`

Not sure which plugins to install? Run `/perimetre-discover:discover`. In Claude Code it silently scans your project filesystem for signals; in Claude Cowork it reads conversation context. Asks at most 3 questions, then recommends the right plugins with copy-pasteable install commands.

**Skills:** `marketplace-catalog`

**Commands:** `/perimetre-discover:discover`

## Updates

Changes pushed to this repository become available to all users immediately. Plugins are git-based — there are no version folders to manage; the latest commit on the default branch is always the current version.

**To update manually:**
```
/plugin marketplace update perimetre-ai-toolkit
```

**To enable auto-update** (off by default for third-party marketplaces):
```
/plugin → Marketplaces → perimetre-ai-toolkit → Enable auto-update
```
When enabled, Claude checks for updates on startup and prompts a restart if anything has changed.

## Structure

- `.claude-plugin/` — Marketplace manifest and configuration
- `plugins/` — Individual plugin packages

## Adding a Plugin

Add a new folder under `plugins/` following the structure of an existing plugin, then register it in `.claude-plugin/marketplace.json`.
