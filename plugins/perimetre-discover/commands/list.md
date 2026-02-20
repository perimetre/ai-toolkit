---
description: "Lists all available plugins in the Périmètre marketplace with descriptions."
allowed-tools: [Read]
---

# Périmètre Marketplace — Plugin List

Read `skills/marketplace-catalog/SKILL.md` for the full plugin catalog, then output the following:

## All Available Plugins

For each plugin in the catalog, output a single block:

```
[plugin-name] — [one-sentence description of what it's for]

  What you get: [comma-separated list of key skills/commands]
  Best for: [target persona in plain language]
```

After listing all plugins, end with:

```
---
Install any or all of them:

/plugin marketplace add https://github.com/perimetre/ai-toolkit

Not sure which ones apply to you? Run /perimetre-discover:discover
```

Keep descriptions short and jargon-free — this command may be read by non-developers.
