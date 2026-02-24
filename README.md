# Périmètre AI Toolkit

**Version:** 0.4.0 — Last updated: 2026-02-24

Périmètre's internal plugin marketplace for Claude. Gives Claude deep knowledge of Périmètre's tools, patterns, and standards — for developers, designers, writers, and project managers.

---

## Installation

### Claude Code

Add the marketplace with one command:

```bash
/plugin marketplace add https://github.com/perimetre/ai-toolkit
```

To update manually when new plugins or improvements are released:

```bash
/plugin marketplace update perimetre-ai-toolkit
```

### Claude Cowork

Cowork has a UI option to add a marketplace from GitHub — use `perimetre/ai-toolkit`. Updates are manual via the "Check for updates" option on the installed marketplace entry.

> Updates in both Claude Code and Cowork are version-based — a new version is only detected if the `version` field in the plugin manifest has been bumped.

Org-wide plugin provisioning is planned by Anthropic but not yet available — installation is per-user.

---

## Plugins

Not sure where to start? Install `perimetre-discover` first — it will recommend the right plugins for your project.

---

### `perimetre-discover` ← Start here if unsure

**Version:** 1.1.0

Plugin discovery assistant. Scans your project or reads conversation context, then recommends the right plugins with plain-language explanations and install commands.

**Audience:** Anyone new to the Périmètre marketplace, or unsure which plugins apply to their work. Works for developers, designers, writers, and non-technical team members.

**Skills:**

- `marketplace-catalog` — Complete catalog of all plugins with target personas, signals, and install commands

**Commands:**

- `/perimetre-discover:discover` — Detects context, asks up to 3 questions if needed, and recommends plugins with reasons
- `/perimetre-discover:list` — Lists all available plugins with descriptions and target personas

---

### `perimetre-apps`

**Version:** 1.1.0

Plugin for JavaScript/TypeScript app development at Périmètre. Covers the full React/Next.js stack — framework patterns, code review, and planning agents.

**Audience:** Developers building apps and prototypes with the Périmètre JS/TS/React/Next.js stack.

**Skills:**

- `error-handling` — Error-as-values pattern using discriminated unions
- `services` — Service layer architecture and `defineService`
- `trpc` — tRPC routers, procedures, and middleware
- `forms` — React Hook Form + Zod, shared schema, i18n errors
- `graphql` — GraphQL with TanStack Query and codegen
- `tanstack-query` — `queryOptions` factory, prefetching, cache invalidation
- `icons` — `@perimetre/icons`, `Icon` wrapper, accessibility
- `images` — Next.js `Image` component, fill, sizes, LCP
- `seo` — `generateMetadata`, Open Graph, JSON-LD, sitemap
- `caching` — Next.js fetch cache, `React.cache`, `unstable_cache`
- `payload-drizzle` — Payload CMS + Drizzle ORM integration
- `code-review` — Multi-agent review with confidence scoring
- `vercel-react-best-practices` — 57 performance rules across 8 categories
- `vercel-composition-patterns` — Compound components, React 19 patterns
- `web-design-guidelines` — UI compliance against Vercel Interface Guidelines
- `web-animation-design` — Easing, timing, springs, `prefers-reduced-motion`
- `emil-design-engineering` — UI polish, touch, accessibility, marketing

**Commands:**

- `/perimetre-apps:code-review` — Multi-agent code review (supports changes, PR, path, and GitHub PR modes)

**Agents:**

- `feature-planner` — Architecture blueprint for a new feature
- `scaffold-advisor` — Minimal idiomatic boilerplate for a pattern combination

---

### `perimetre-brand-guidelines`

**Version:** 1.0.0

Loads Périmètre's visual identity into Claude's context — colours, typography, and logo usage — so every deliverable is on-brand.

**Audience:** Anyone producing deliverables that carry the Périmètre name: developers, designers, writers, and project managers. Lightweight — install it alongside any other plugin.

**Skills:**

- `brand-guidelines` — Colour palette (Carbon `#111111`, Steel `#737678`, Aqua `#13FADC`), Satoshi typeface rules, logo usage, tone of voice

**Commands:** None — the skill activates automatically for brand-related content.

---

### `perimetre-design-system`

**Version:** 1.1.0

Plugin for developers building or extending `@perimetre/ui` — Périmètre's brand-aware React component library built on Tailwind CSS v4, CVA, and Radix UI.

**Audience:** Developers working **on** the `@perimetre/ui` package itself. Not for apps that consume the library — use `perimetre-apps` for that.

**Skills:**

- `design-system` — Three-tier token architecture, CVA brand variants, `compose()` patterns, accessibility, RSC compatibility. Includes 11 detailed rule files.

**Commands:**

- `/perimetre-design-system:code-review` — Multi-agent code review for design system component work

**Agents:**

- `component-planner` — Architecture blueprint for a new component
- `token-impact-analyzer` — Scans for breaking changes when modifying a design token

---

### `perimetre-documents`

**Version:** 1.0.0

Plugin for professional document writing at Périmètre. Covers structure, clarity, tone, and formatting for reports, proposals, and presentations.

**Audience:** Anyone on the Périmètre team writing professional documents — developers writing specs, project managers, or non-technical team members producing client-facing deliverables. Pair with `perimetre-brand-guidelines` for on-brand output.

**Skills:**

- `document` — Leading with the conclusion, one-idea-per-section, heading hierarchy, concise prose, client-facing tone, executive summary structure

**Commands:** None — the skill activates automatically when Claude writes or reviews a document.

---

### `perimetre-wordpress`

**Version:** 1.2.0

Plugin for WordPress development at Périmètre. Covers PHP coding standards, security, Gutenberg blocks (native and ACF Pro), WPGraphQL, WPML, REST API extensions, and i18n.

**Audience:** Developers building WordPress sites — PHP themes, Gutenberg blocks, custom plugins, and headless WordPress with Next.js.

**Skills:**

- `code-review` — WordPress-specific review: WPCS, security, hooks, Gutenberg, i18n, query performance
- `wordpress-patterns` — Theme structure, CPTs, block development with `@wordpress/scripts`, ACF Pro, WPGraphQL, WPML, REST API, headless WP + Next.js

**Commands:**

- `/perimetre-wordpress:code-review` — Multi-agent review across 5 parallel specialized agents (security, WPCS, Gutenberg, hooks, i18n/performance)

**Agents:**

- `block-planner` — Architecture blueprint for a new Gutenberg block
- `cpt-taxonomy-planner` — CPT and taxonomy registration, WPGraphQL, WPML strategy

---

### `perimetre-a11y`

**Version:** 1.1.0

Automated accessibility auditing for web properties. Crawls with Playwright, runs pa11y/axe-core/Lighthouse, maps findings to WCAG 2.2 and Canadian jurisdiction law, and produces a prioritized report.

**Audience:** Developers and QA engineers who need to audit web properties for WCAG 2.2 compliance and Canadian legal requirements (Federal ACA, Ontario AODA, Quebec SGQRI/Charter). Works in degraded mode when Playwright or scanners are unavailable.

**Skills:**

- `wcag-standards` — POUR principles, Level A/AA SC reference, common failure patterns, DOM inspection checklist (13 checks), scoring formula
- `canadian-accessibility-law` — Federal/Ontario/Quebec jurisdiction table, SC-level legal signal lookup, citation formats, multi-jurisdiction guidance

**Commands:**

- `/perimetre-a11y:audit <url> [--jurisdiction global|federal|ontario|quebec] [--depth N]` — Full 4-stage pipeline: crawl → scan → map → report

**Agents:**

- `web-crawler` — BFS Playwright crawler with 13 per-page DOM checks (gracefully degrades if Playwright unavailable)
- `scanner-runner` — pa11y / axe-core / Lighthouse runner, best-effort (degrades if scanners not installed)
- `standards-mapper` — Normalizes, deduplicates, scores 0–100, adds law citations
- `report-writer` — Produces structured HIGH/MEDIUM/LOW Markdown report

---

## Recommended stacks

| Context | Install these |
|---------|---------------|
| Front-end / full-stack JS app | `perimetre-apps` + `perimetre-brand-guidelines` |
| Design system work | `perimetre-apps` + `perimetre-design-system` + `perimetre-brand-guidelines` |
| WordPress site | `perimetre-wordpress` + `perimetre-brand-guidelines` |
| Headless WP + Next.js | `perimetre-wordpress` + `perimetre-apps` |
| Document / writing work | `perimetre-documents` + `perimetre-brand-guidelines` |
| Accessibility audit | `perimetre-a11y` |
| Everything | Install all — they don't conflict |

---

## For maintainers

- **No build step** — committing to `main` is the deployment. Changes are available to all users after they manually update (`/plugin marketplace update` in Claude Code; "Check for updates" in Cowork). **Always bump the `version` field in `plugin.json`** — both clients skip updates if the version hasn't changed.
- **Adding a plugin** — create `plugins/[plugin-name]/` following the structure of an existing plugin, then register it in `.claude-plugin/marketplace.json`.
- **Repository layout** — `.claude-plugin/` holds the marketplace manifest; `plugins/` holds the individual plugin packages.
