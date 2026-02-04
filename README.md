# AI Toolkit

A plugin marketplace for Claude Code and other AI tools.

## Installation

Add this marketplace to Claude Code:

```bash
/plugin marketplace add ./ai-toolkit
```

## Structure

- `.claude-plugin/` - Plugin manifest and marketplace configuration
- `plugins/` - Individual plugin packages

## Adding Plugins

Plugins are added to the `plugins/` directory. Each plugin should follow the Claude Code plugin specification.

## Validation

Validate the marketplace structure:

```bash
claude plugin validate .
```
