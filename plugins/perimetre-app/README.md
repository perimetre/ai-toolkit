# perimetre-app

Claude plugin for Périmètre app development. Covers the full stack of patterns used in JS/TS/React/Next.js projects.

## Audience

Dev team (and general team) building apps and prototypes with the Périmètre framework.

## Skills

### Framework Patterns

Migrated from `framework/LLMs/` — each skill covers one technology area in detail.

| Skill | Description |
|-------|-------------|
| `error-handling` | Error-as-values pattern, custom error classes, ok/error returns |
| `services` | Service layer architecture, `defineService`, route delegation |
| `trpc` | tRPC routers, procedures, middleware, and Périmètre conventions |
| `forms` | React Hook Form + Zod, single shared schema, i18n error messages |
| `graphql` | GraphQL with TanStack Query, codegen, fragment usage |
| `tanstack-query` | `queryOptions` factory pattern, prefetching, cache invalidation |
| `icons` | `@perimetre/icons`, Icon wrapper, `currentColor`, accessibility |
| `images` | Next.js Image component, fill, sizes, alt text, LCP priority |
| `seo` | `generateMetadata`, Open Graph, JSON-LD, sitemap, robots.txt |
| `caching` | Next.js fetch cache, `React.cache`, `unstable_cache`, revalidation |
| `payload-drizzle` | Payload CMS + Drizzle ORM shared instance and migrations |

### Code Review

| Skill | Description |
|-------|-------------|
| `code-review` | Multi-agent PR review with confidence scoring and local pattern references |

### React & Performance (third-party, migrated as-is)

| Skill | Description |
|-------|-------------|
| `vercel-react-best-practices` | 57 Vercel-authored performance rules across 8 categories |
| `vercel-composition-patterns` | Compound components, boolean prop avoidance, React 19 patterns |

### Design & UX (third-party, migrated as-is)

| Skill | Description |
|-------|-------------|
| `web-design-guidelines` | UI compliance review against Vercel Web Interface Guidelines |
| `web-animation-design` | Easing, timing, springs, performance, `prefers-reduced-motion` |
| `emil-design-engineering` | UI polish, forms, touch, accessibility, marketing, performance |
