---
description: >
  Helps you figure out which Périmètre plugins to install.
  Scans your project context and asks a few questions, then recommends
  the right plugins with plain-language explanations and install commands.
allowed-tools: [Glob, Grep, Bash, Read]
---

# Périmètre Plugin Discovery

Help the user find the right Périmètre plugins for their context. Follow these steps precisely.

## Step 0: Load Plugin Knowledge

Before doing anything else, read the marketplace catalog skill:

```
skills/marketplace-catalog/SKILL.md
```

This file contains the complete, authoritative catalog of all plugins — their signals, what they provide, and installation commands. All recommendations must come from this file.

## Step 1: Detect Environment + Gather Signals

**Detect which environment you are in first:**

- **Claude Code** — filesystem tools are available; the user is likely in a project directory
- **Claude Cowork** — no filesystem access; rely on conversation context

### In Claude Code (filesystem available)

Run silent filesystem checks. Produce **no output yet** — just collect signals internally.

Check in parallel:

1. **WordPress signals:**
   - `Glob` for `**/functions.php`, `**/wp-config.php`, `**/style.css`
   - `Glob` for `wp-content/` directory presence
   - `Bash`: `ls` to see top-level directory structure

2. **Apps signals:**
   - `Glob` for `**/package.json`
   - If found: `Grep` in `package.json` for `"next"`, `"react"`, `"@trpc"`, `"@tanstack"`

3. **Design system signals:**
   - `Grep` in source files (`**/*.ts`, `**/*.tsx`) for `@perimetre/ui`
   - `Grep` for `cva(` or `data-pui-brand`

4. **Document/content signals:**
   - `Glob` for `**/*.md`, `**/*.docx`
   - Check if code files are absent (no `.ts`, `.tsx`, `.js`, `.php`)

Collect all signals. If the directory is empty or signals are ambiguous (e.g. only a `package.json` with no recognizable deps), proceed to Step 2 to ask.

### In Claude Cowork (no filesystem)

Read the conversation history for signals:
- User pasted a document, is asking about writing, or mentions proposals/reports → Documents + Brand signal
- User mentions UI, shares a Figma link, or asks about building a prototype → Apps signal (+ possibly Design System)
- User mentions WordPress, a CMS, or a website → WordPress signal
- User mentions Périmètre colours, logo, or visual identity → Brand Guidelines signal
- User says "vibe coding", "prototype", or "build something fast" → Apps signal
- No clear signals → proceed to Step 2

## Step 2: Ask Targeted Questions (1–3 max)

Only ask what context hasn't already answered. Keep it conversational — one or two short questions, not a form.

**Examples:**

- If no signals at all: "What are you mainly working on? Building an app or website, working with WordPress, writing a document, or something else?"
- If some dev signals but unclear stack: "Are you using Périmètre's design system (`@perimetre/ui`) in this project?"
- If unclear dev vs. content context: "Is this primarily development work or content/writing work?"

**Stop at 3 questions maximum.** If still uncertain after 3, make your best recommendation with qualifiers ("if you're working with `@perimetre/ui`, also add…").

## Step 3: Generate Recommendations

For each recommended plugin, output a card using this exact format:

```
──────────────────────────────────────
[plugin-name]

Why it fits: [1 sentence grounded in the detected context — e.g. "Found next and @trpc/client in your package.json."]

What you get:
• [Key command or feature 1]
• [Key command or feature 2]
• [Key command or feature 3]
──────────────────────────────────────
```

Apply recommendation tiers from the catalog:

- **Dev project (JS/TS signals):** Lead with `perimetre-apps`. Surface `perimetre-brand-guidelines` as "also useful if the project should match Périmètre's visual identity." Surface `perimetre-design-system` only if `@perimetre/ui` signals were found or confirmed.
- **WordPress signals:** Lead with `perimetre-wordpress`. Surface `perimetre-apps` as "also useful if there's a headless Next.js front-end."
- **No code signals / Cowork writing context:** Lead with `perimetre-documents` + `perimetre-brand-guidelines` as a natural pair. Mention dev plugins only if the user indicated they do development work.
- **Mixed signals:** Show all relevant plugins with clear "if X" qualifiers.

## Step 4: Install Instructions

After the recommendation cards, always output this section:

```
## Install

All Périmètre plugins are installed from the same marketplace:

/plugin marketplace add https://github.com/perimetre/ai-toolkit

After installing, enable only the plugins you need from your Claude settings —
or enable all of them. They don't conflict.
```

Then, if multiple plugins were recommended, add a brief stack summary:

```
## Recommended Stack

[Plugin 1] + [Plugin 2] [+ Plugin 3 if applicable] — [one sentence on how they work together]
```

---

## Important Notes

- **Do not auto-install.** The `/plugin marketplace add` command cannot be invoked by Claude — it can only be surfaced for the user to copy-paste. This is a platform constraint.
- **Install command is always the same URL** — `https://github.com/perimetre/ai-toolkit` — regardless of which plugins are recommended.
- **All plugin knowledge comes from the catalog skill.** Do not invent plugin details.
- **Stay conversational.** This command may run for non-developers — avoid technical jargon unless the user is clearly a developer.
- **Silent scan, then ask.** In Claude Code, always scan the filesystem silently before asking any questions. Never ask something the filesystem already answered.
