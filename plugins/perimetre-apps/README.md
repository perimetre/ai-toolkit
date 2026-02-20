# perimetre-apps

**Version:** 1.1.0 — Last updated: 2026-02-19

Plugin for JavaScript/TypeScript app development at Périmètre. Covers framework patterns, code review, and planning agents for the full React/Next.js stack.

## Audience

Developers building apps and prototypes with the Périmètre JS/TS/React/Next.js stack. Also useful for anyone learning Périmètre's technical conventions or doing design engineering work.

## Skills

### Framework Patterns

| Skill | What it covers |
|-------|----------------|
| `error-handling` | Error-as-values pattern using discriminated unions — custom error classes, `ok`/`error` returns, no thrown exceptions |
| `services` | Service layer architecture — `defineService`, route delegation, separation of concerns |
| `trpc` | tRPC routers, procedures, middleware, and Périmètre conventions |
| `forms` | React Hook Form + Zod — single shared schema, field validation, i18n error messages |
| `graphql` | GraphQL with TanStack Query — codegen, fragment usage, typed responses |
| `tanstack-query` | `queryOptions` factory pattern, prefetching, cache invalidation |
| `icons` | `@perimetre/icons`, `Icon` wrapper, `currentColor`, accessibility |
| `images` | Next.js `Image` component — fill, sizes, alt text, LCP priority |
| `seo` | `generateMetadata`, Open Graph, JSON-LD, sitemap, robots.txt |
| `caching` | Next.js fetch cache, `React.cache`, `unstable_cache`, revalidation strategies |
| `payload-drizzle` | Payload CMS + Drizzle ORM shared instance and migrations |
| `code-review` | Multi-agent code review with confidence scoring and local pattern references |

### React & Performance

| Skill | What it covers |
|-------|----------------|
| `vercel-react-best-practices` | 57 Vercel-authored performance rules across 8 categories |
| `vercel-composition-patterns` | Compound components, boolean prop avoidance, React 19 patterns |

### Design & UX

| Skill | What it covers |
|-------|----------------|
| `web-design-guidelines` | UI compliance review against Vercel Web Interface Guidelines |
| `web-animation-design` | Easing, timing, springs, performance, `prefers-reduced-motion` |
| `emil-design-engineering` | UI polish, forms, touch, accessibility, marketing, performance |

## Commands

| Command | Description |
|---------|-------------|
| `/perimetre-apps:code-review` | Multi-agent code review for React/Next.js projects |

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
| `feature-planner` | Planning a new feature | Architecture blueprint: service boundaries, error handling strategy, data fetching, tRPC procedures, cache invalidation, form structure |
| `scaffold-advisor` | Starting implementation | Minimal idiomatic boilerplate for a specific Périmètre pattern combination (e.g. tRPC mutation + form + optimistic update) |
