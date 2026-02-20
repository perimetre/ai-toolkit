---
name: marketplace-catalog
description: >
  Complete, structured knowledge of every plugin in the Périmètre marketplace —
  what each plugin is, who it's for, what it contains, and how to install it.
  Used by the discover command to make accurate recommendations without hallucinating.
---

# Périmètre Marketplace Catalog

This is the authoritative reference for all plugins available in the Périmètre marketplace. Use this catalog to make accurate plugin recommendations. Never guess or invent plugin details — consult this file.

## Marketplace Install Command

**Claude Code:**
```
/plugin marketplace add https://github.com/perimetre/ai-toolkit
```

**Claude Cowork:** There is a UI option to add a marketplace from GitHub using `perimetre/ai-toolkit`. Updates are manual via the "Check for updates" option on the installed marketplace entry.

Note: both environments detect updates via the `version` field in `plugin.json` — a content change without a version bump will not be picked up.

After installing, all plugins are available. Users enable or disable individual plugins from their Claude settings. All plugins coexist without conflict.

---

## Plugin Catalog

### 1. `perimetre-apps`

**Target persona:** Developers building JavaScript/TypeScript applications at Périmètre — React, Next.js, tRPC, TanStack Query, forms, GraphQL.

**What it provides:**
- `/perimetre-apps:code-review` — multi-agent code review for React/Next.js projects (supports changes, PR, path, and GitHub PR modes)
- Planning agents for new features and components
- 13+ skills covering: error handling (Go/Rust-style discriminated unions), service patterns, tRPC, forms, TanStack Query, GraphQL, caching, SEO, icons, images, Payload CMS + Drizzle, Vercel deployment rules, React composition patterns, design engineering guidelines, web animation

**Project signals (install if any of these are present):**
- `package.json` containing `next`, `react`, `@trpc/client`, `@tanstack/react-query`
- Source files importing from `@/lib/`, `@/components/`, `@/hooks/`
- Files with `.tsx` or `.ts` extensions
- User mentions building an app, a prototype, a Next.js site, or "vibing" something

**When to install:** Any front-end or full-stack JavaScript/TypeScript project. This is the core dev plugin — install it first for any JS/TS work.

**When to skip:** Pure WordPress PHP projects with no JS app layer, or non-dev content/writing work.

**Key commands after installing:**
- `/perimetre-apps:code-review` — review uncommitted changes
- `/perimetre-apps:code-review --pr 123` — review a pull request
- Invoke any skill by name (e.g. "use the error-handling skill")

---

### 2. `perimetre-brand-guidelines`

**Target persona:** Anyone producing deliverables that represent Périmètre — developers, designers, writers, project managers.

**What it provides:**
- `brand-guidelines` skill — loads Périmètre's complete visual identity into context: primary colours (Carbon `#111111`, Aqua `#13FADC`), accent palette, typography (Satoshi typeface), logo usage rules, spacing, and tone of voice

**Project signals (install if any of these are present):**
- User is producing a document, proposal, or presentation for a client
- User references Périmètre colours, logo, or brand
- User is building a UI that should match Périmètre's visual identity
- Presence of brand assets (`logo.svg`, `brand/`, colour variables referencing Carbon or Aqua)

**When to install:** Almost always useful alongside other plugins. It's lightweight and never conflicts. Pairs especially well with `perimetre-documents` (brand-aware writing) and `perimetre-apps` (on-brand UI work).

**When to skip:** Internal tooling with no visual brand requirements, or third-party client projects that use their own brand.

**Key commands after installing:**
- No dedicated command — the skill activates automatically when Claude writes brand-related content

---

### 3. `perimetre-design-system`

**Target persona:** Developers implementing or extending the `@perimetre/ui` component library.

**What it provides:**
- `design-system` skill — comprehensive implementation guide for `@perimetre/ui`: CVA (Class Variance Authority) patterns, three-tier design token system, multi-brand CSS with `data-pui-brand`, Tailwind CSS v4 integration, Radix UI primitives, accessibility patterns
- `/perimetre-design-system:code-review` — code review specialized for design system component work
- 11 supplementary rule files covering token architecture and brand system details
- Planning and development agents for design system components

**Project signals (install if any of these are present):**
- `package.json` containing `@perimetre/ui`
- Source files importing from `@perimetre/ui`
- CVA usage (`cva(`, `VariantProps`)
- `data-pui-brand` attributes in JSX
- Working in a `packages/ui/` or `src/components/ui/` directory

**When to install:** Only when actively working with `@perimetre/ui`. This is a specialized plugin — install `perimetre-apps` first.

**When to skip:** Projects that don't use `@perimetre/ui`, or projects where you're consuming (not building) design system components with no custom extensions needed.

**Key commands after installing:**
- `/perimetre-design-system:code-review` — review design system component changes
- Invoke the skill by name for implementation guidance

---

### 4. `perimetre-wordpress`

**Target persona:** PHP developers building WordPress themes, plugins, or headless WordPress integrations at Périmètre.

**Project signals (install if any of these are present):**
- `functions.php`, `style.css` (theme header), or `wp-config.php` present
- Directories: `wp-content/`, `wp-includes/`, `plugins/`, `themes/`
- PHP files with WordPress function calls (`add_action(`, `get_posts(`, `register_post_type(`)
- `composer.json` with WordPress-related packages
- User mentions WordPress, WooCommerce, ACF, WPGraphQL, WPML, Gutenberg blocks

**What it provides:**
- `/perimetre-wordpress:code-review` — multi-agent code review covering: security (nonces, capability checks, sanitization, escaping), PHP Coding Standards (WPCS), Gutenberg block patterns, REST API, i18n compliance, and query performance
- `wordpress-patterns` skill — best practices for PHP themes, custom post types, Gutenberg blocks with `@wordpress/scripts`, REST API extensions, and headless WP + Next.js integration
- Specialized review agents: security-reviewer, gutenberg-reviewer, hooks-history-reviewer, wpcs-reviewer, i18n-performance-reviewer

**When to install:** Any WordPress PHP project. Not needed for headless WP projects where all code is on the JS side (use `perimetre-apps` instead).

**When to skip:** Non-WordPress PHP, or JS-only projects consuming a WordPress REST/GraphQL API.

**Key commands after installing:**
- `/perimetre-wordpress:code-review` — review uncommitted changes
- `/perimetre-wordpress:code-review --pr 123` — review a PR

---

### 5. `perimetre-documents`

**Target persona:** Anyone on the Périmètre team writing professional documents — developers writing specs, project managers writing proposals, anyone producing reports or presentations.

**What it provides:**
- `document` skill — professional writing principles: leading with the conclusion, one idea per section, heading hierarchy, table usage, concise prose, client-facing tone. Brand-agnostic by default; pairs with `perimetre-brand-guidelines` for on-brand output.

**Project signals (install if any of these are present):**
- No code files present — primarily Markdown, Word docs, or text
- User is in Claude Cowork (not Claude Code) and asking about writing
- User is drafting a proposal, report, brief, or presentation
- User asks for help structuring a document

**When to install:** For anyone doing professional writing work, especially in Claude Cowork. This is the default plugin for non-dev Cowork users.

**When to skip:** Pure development work with no document deliverables.

**Key commands after installing:**
- No dedicated command — the skill activates when Claude writes or reviews documents

---

## Recommended Stacks

### Full front-end development stack
All three of these work together seamlessly:
- `perimetre-apps` — core dev patterns
- `perimetre-brand-guidelines` — visual identity
- `perimetre-design-system` — if using `@perimetre/ui`

Install all three, or just `perimetre-apps` to start and add the others as needed.

### WordPress development stack
- `perimetre-wordpress` — PHP and WordPress patterns
- `perimetre-apps` — if there's a headless Next.js front-end alongside
- `perimetre-brand-guidelines` — if producing brand-aligned output

### Cowork / writing stack (non-developer)
- `perimetre-documents` — document structure and clarity
- `perimetre-brand-guidelines` — on-brand output

This is the default recommendation for Cowork users with no code signals.

### Prototype / vibe coding stack
- `perimetre-apps` — always recommended; covers Next.js, React, and rapid prototyping patterns
- `perimetre-brand-guidelines` — surface this if the project uses Périmètre's visual identity
- `perimetre-design-system` — surface this only if the user is working with `@perimetre/ui` specifically

Do not assume a vibe project uses `@perimetre/ui` — ask or check for signals first.

---

## Important Notes for the discover Command

- **Never auto-install.** The `/plugin marketplace add` command is a slash command and cannot be invoked programmatically. Always surface the exact command for the user to copy-paste.
- **All plugins are installed from the same URL** — `https://github.com/perimetre/ai-toolkit` — and enabled/disabled individually.
- **All plugins coexist without conflict.** Installing extras has no downside beyond context size.
- **When signals are mixed**, lean toward recommending more plugins with clear "if X applies to you" qualifiers rather than under-recommending.
- **For Cowork users with no code signals**, always lead with `perimetre-documents` + `perimetre-brand-guidelines` before mentioning dev plugins.
