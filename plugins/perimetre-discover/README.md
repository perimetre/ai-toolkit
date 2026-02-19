# perimetre-discover

**Version:** 1.0.0 — Last updated: 2026-02-19

Plugin discovery assistant for the Périmètre marketplace. Scans your project context (in Claude Code) or reads conversational cues (in Claude Cowork), asks at most a few targeted questions, then recommends the right plugins with plain-language explanations and copy-pasteable install commands.

## When to use it

Run `/perimetre-discover:discover` when you're not sure which Périmètre plugins apply to your project or workflow.

## Command

| Command | Description |
|---------|-------------|
| `/perimetre-discover:discover` | Detect context, ask up to 3 questions, and recommend plugins |

## How it works

### In Claude Code
1. **Silent filesystem scan** — checks for `package.json`, WordPress files, `@perimetre/ui` imports, and other signals without asking questions
2. **Targeted questions** (only if signals are ambiguous) — at most 1–3 short questions
3. **Recommendation cards** — one card per recommended plugin with a reason, key features, and the install command

### In Claude Cowork
1. **Reads conversation context** — infers signals from what you've said (documents, UI, WordPress, brand)
2. **Targeted questions** (if needed) — same lightweight approach
3. **Recommendation cards** — same output format

## Output example

```
──────────────────────────────────────
perimetre-apps

Why it fits: Found next and @trpc/client in your package.json.

What you get:
• Code review command: /perimetre-apps:code-review
• Planning agents for new features and components
• 13+ skills covering error handling, tRPC, forms, data fetching...
──────────────────────────────────────

## Install

All Périmètre plugins are installed from the same marketplace:

/plugin marketplace add https://github.com/perimetre/ai-toolkit

After installing, enable only the plugins you need from your Claude settings —
or enable all of them. They don't conflict.
```

## Design notes

- **No auto-install** — `/plugin marketplace add` is a slash command; Claude cannot invoke it on your behalf. The command is always surfaced as copy-pasteable output.
- **Dual-environment** — works in both Claude Code (filesystem scan first) and Claude Cowork (conversation context first).
- **3 questions max** — non-devs abandon flows that feel like forms. The command is intentionally fast.
- **Single source of truth** — all plugin knowledge lives in `skills/marketplace-catalog/SKILL.md`. When a new plugin is added to the marketplace, only that file needs updating.

## Skill

| Skill | Description |
|-------|-------------|
| `marketplace-catalog` | Complete catalog of all marketplace plugins with signals, features, and install commands |
