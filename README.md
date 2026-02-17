# AI Toolkit

Périmètre's internal plugin marketplace for Claude.

## Installation

Add this marketplace to Claude:

```
/plugin marketplace add https://github.com/perimetre/ai-toolkit
```

## Plugins

### `perimetre-brand`

Loads Périmètre's visual identity guidelines into context — colours, typography, and logo usage — so every deliverable is on-brand.

**Skills:** `brand-guidelines`

## Structure

- `.claude-plugin/` — Marketplace manifest and configuration
- `plugins/` — Individual plugin packages

## Adding a Plugin

Add a new folder under `plugins/` following the structure of an existing plugin, then register it in `.claude-plugin/marketplace.json`.
