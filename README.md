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

## Updates

Changes pushed to this repository become available to all users immediately. Plugins are git-based — there are no version folders to manage; the latest commit on the default branch is always the current version.

**To update manually:**
```
/plugin marketplace update ai-toolkit
```

**To enable auto-update** (off by default for third-party marketplaces):
```
/plugin → Marketplaces → ai-toolkit → Enable auto-update
```
When enabled, Claude checks for updates on startup and prompts a restart if anything has changed.

## Structure

- `.claude-plugin/` — Marketplace manifest and configuration
- `plugins/` — Individual plugin packages

## Adding a Plugin

Add a new folder under `plugins/` following the structure of an existing plugin, then register it in `.claude-plugin/marketplace.json`.
