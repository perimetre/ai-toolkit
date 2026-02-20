# perimetre-discover

**Version:** 1.1.0 — Last updated: 2026-02-20

Plugin discovery assistant for the Périmètre marketplace. Scans your project context or reads conversational cues, then recommends the right plugins with plain-language explanations and copy-pasteable install commands.

## Audience

Anyone installing Périmètre plugins for the first time, or anyone unsure which plugins apply to their current project or workflow. Works for developers, designers, writers, and non-technical team members alike.

## Skills

| Skill | What it covers |
|-------|----------------|
| `marketplace-catalog` | Complete catalog of all marketplace plugins — target personas, project signals, what each plugin provides, recommended stacks, and installation commands. Single source of truth for the `discover` and `list` commands. |

## Commands

| Command | Description |
|---------|-------------|
| `/perimetre-discover:discover` | Detects your project context, asks up to 3 questions if needed, then recommends plugins with reasons and install commands |
| `/perimetre-discover:list` | Lists all available plugins with plain-language descriptions and target personas |

**How `/discover` works:**

- **In Claude Code** — silently scans the filesystem for signals (WordPress files, `package.json` deps, `@perimetre/ui` imports) before asking any questions
- **In Claude Cowork** — reads conversation context for signals (documents, UI descriptions, WordPress mentions, brand references)
- In both environments: asks at most 3 short questions if signals are ambiguous, then outputs recommendation cards with copy-pasteable install commands

> **Note:** `/plugin marketplace add` is a slash command — Claude cannot invoke it on your behalf. The install command is always surfaced for you to copy-paste.
