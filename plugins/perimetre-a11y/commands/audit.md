---
description: "Automated accessibility audit: crawls a web property with Playwright, runs pa11y/axe/Lighthouse, maps findings to WCAG 2.2 and Canadian law, and produces a prioritized HIGH/MEDIUM/LOW report."
argument-hint: "<url> [--jurisdiction global|federal|ontario|quebec] [--depth N] [--lang en|fr]"
allowed-tools: [Bash, Task, mcp__playwright__browser_navigate, mcp__playwright__browser_snapshot, mcp__playwright__browser_evaluate, mcp__playwright__browser_close, mcp__playwright__browser_install, mcp__plugin_playwright_playwright__browser_navigate, mcp__plugin_playwright_playwright__browser_snapshot, mcp__plugin_playwright_playwright__browser_evaluate, mcp__plugin_playwright_playwright__browser_close, mcp__plugin_playwright_playwright__browser_install]
---

# Périmètre Accessibility Audit

Run a full automated accessibility audit on a web property.

## Arguments

| Argument | Default | Description |
|----------|---------|-------------|
| `<url>` | _(required)_ | The seed URL to audit (must be http:// or https://) |
| `--jurisdiction` | `global` | Legal jurisdiction: `global`, `federal`, `ontario`, or `quebec` |
| `--depth N` | `8` | BFS crawl depth, clamped to 1–20 |
| `--lang` | `en` | Report language: `en` (English) or `fr` (French) |

---

## Step 0: Parse and Validate Arguments

Parse the arguments provided to this command:

1. **Extract URL** — the first non-flag argument. Required. If missing, output:
   ```
   Error: URL is required.
   Usage: /perimetre-a11y:audit <url> [--jurisdiction global|federal|ontario|quebec] [--depth N] [--lang en|fr]
   ```
   and stop.

2. **Validate URL** — must start with `http://` or `https://`. If not, output:
   ```
   Error: URL must start with http:// or https://
   ```
   and stop.

3. **Extract jurisdiction** — look for `--jurisdiction <value>` flag. Valid values: `global`, `federal`, `ontario`, `quebec`. Default: `global`. If an invalid value is provided, default to `global` and warn.

4. **Extract depth** — look for `--depth <N>` flag. Parse as integer. Clamp to range 1–20. Default: `8`.

5. **Extract language** — look for `--lang <value>` flag. Valid values: `en`, `fr`. Default: `en`. If an invalid value is provided, default to `en` and warn.

6. **Derive audit date** — use today's date in ISO format (YYYY-MM-DD).

Echo the parsed configuration to the user before proceeding:

```
Accessibility Audit
===================
URL:          [url]
Jurisdiction: [jurisdiction]
Depth:        [depth]
Language:     [lang]
Date:         [audit_date]

Starting pipeline...
```

---

## Step 1: Crawl

Use the `web-crawler` agent via Task to crawl the web property.

Pass in the prompt:
```
Crawl the following web property for accessibility DOM issues.

URL: [url]
DEPTH: [depth]
JURISDICTION: [jurisdiction]

Apply the 13 DOM checks from the wcag-standards skill to every page you successfully visit.
```

Store the full agent output as `CRAWLER_OUTPUT`.

Extract from `CRAWLER_OUTPUT`:
- `PLAYWRIGHT_STATUS` (look for `PLAYWRIGHT_STATUS:` line)
- `PAGE_INVENTORY` JSON array (content between `PAGE INVENTORY` and `DOM FINDINGS` sections)
- `DOM_FINDINGS` JSON array (content between `DOM FINDINGS` and `CRAWL NOTES` sections)
- `PAGES_CRAWLED` count
- `CRAWL_NOTES` text

Inform the user: `Step 1 complete — [PAGES_CRAWLED] pages crawled (Playwright: [PLAYWRIGHT_STATUS])`

---

## Step 2: Scan

Use the `scanner-runner` agent via Task to run CLI accessibility scanners.

Pass in the prompt:
```
Run accessibility scanners against the following page inventory.

PAGE_INVENTORY:
[PAGE_INVENTORY JSON]

SEED_URL: [url]
JURISDICTION: [jurisdiction]

Run pa11y, axe, and Lighthouse best-effort for up to 20 URLs.
```

Store the full agent output as `SCANNER_OUTPUT`.

Extract from `SCANNER_OUTPUT`:
- `SCANNER_STATUS` (look for `SCANNER_STATUS:` line)
- `PA11Y_STATUS`, `AXE_STATUS`, `LIGHTHOUSE_STATUS`
- `SCANNER_FINDINGS` JSON array
- `SCANNER_NOTES` text

Inform the user: `Step 2 complete — scanners: pa11y=[PA11Y_STATUS] axe=[AXE_STATUS] lighthouse=[LIGHTHOUSE_STATUS]`

---

## Step 3: Map

Use the `standards-mapper` agent via Task to normalize, deduplicate, score, and add jurisdiction citations.

Pass in the prompt:
```
Normalize and score the following accessibility findings.

DOM_FINDINGS:
[DOM_FINDINGS JSON]

SCANNER_FINDINGS:
[SCANNER_FINDINGS JSON]

PAGE_INVENTORY:
[PAGE_INVENTORY JSON]

PLAYWRIGHT_STATUS: [PLAYWRIGHT_STATUS]
SCANNER_STATUS: [SCANNER_STATUS]
JURISDICTION: [jurisdiction]

Apply the full deduplication, scoring formula, and jurisdiction citation logic from your skills.
```

Store the full agent output as `MAPPER_OUTPUT`.

Extract from `MAPPER_OUTPUT`:
- `MAPPED_ISSUES` JSON array
- `TOTAL_ISSUES`, `HIGH_COUNT`, `MEDIUM_COUNT`, `LOW_COUNT` counts
- `MAPPING_NOTES` text

Inform the user: `Step 3 complete — [TOTAL_ISSUES] issues mapped (HIGH: [HIGH_COUNT], MEDIUM: [MEDIUM_COUNT], LOW: [LOW_COUNT])`

---

## Short-Circuit: No Issues Found

If `TOTAL_ISSUES` is 0, output the no-findings message in the chosen language and stop. Do not launch the report-writer agent.

**English (`LANG: en`):**
```
No accessibility issues detected across [PAGES_CRAWLED] page(s).

Scanners: [list available scanners]
Crawl depth: [depth]
Jurisdiction: [jurisdiction]

[If PLAYWRIGHT_STATUS is playwright-unavailable, add:]
Note: DOM-level checks were not performed (Playwright unavailable). Install the Playwright MCP
server for more thorough analysis.

[If SCANNER_STATUS is no-scanners, add:]
Note: No CLI scanners were available. Install pa11y, axe-cli, and lighthouse for scanner coverage.
```

**French (`LANG: fr`):**
```
Aucun problème d'accessibilité détecté sur [PAGES_CRAWLED] page(s).

Scanners : [list available scanners]
Profondeur d'exploration : [depth]
Juridiction : [jurisdiction]

[If PLAYWRIGHT_STATUS is playwright-unavailable, add:]
Note : Les vérifications DOM n'ont pas pu être effectuées (Playwright non disponible). Installez le
serveur MCP Playwright pour une analyse DOM complète.

[If SCANNER_STATUS is no-scanners, add:]
Note : Aucun scanner CLI n'était disponible. Installez pa11y, axe-cli et lighthouse pour une
couverture scanner.
```

---

## Step 4: Report

Use the `report-writer` agent via Task to produce the final Markdown report.

Pass in the prompt:
```
Write a full accessibility audit report from the following data.

MAPPED_ISSUES:
[MAPPED_ISSUES JSON]

PAGE_INVENTORY:
[PAGE_INVENTORY JSON]

PLAYWRIGHT_STATUS: [PLAYWRIGHT_STATUS]
SCANNER_STATUS: [SCANNER_STATUS]
PA11Y_STATUS: [PA11Y_STATUS]
AXE_STATUS: [AXE_STATUS]
LIGHTHOUSE_STATUS: [LIGHTHOUSE_STATUS]
SEED_URL: [url]
JURISDICTION: [jurisdiction]
DEPTH: [depth]
AUDIT_DATE: [audit_date]
PAGES_CRAWLED: [PAGES_CRAWLED]
LANG: [lang]

[If any degradation occurred, include:]
CRAWL_NOTES: [CRAWL_NOTES]
SCANNER_NOTES: [SCANNER_NOTES]
MAPPING_NOTES: [MAPPING_NOTES]
```

Display the report-writer agent's output directly to the user as the final result.

---

## Degraded-Mode Summary

If both `PLAYWRIGHT_STATUS` is `playwright-unavailable` AND `SCANNER_STATUS` is `no-scanners`, prepend the following before the report in the chosen language:

**English (`LANG: en`):**
```
WARNING: Full degraded mode — neither Playwright nor any CLI scanner was available.
This audit could not perform any automated checks. To run a full audit:
1. Configure the Playwright MCP server (see https://code.claude.com/docs)
2. Install accessibility scanners: npm install -g pa11y axe-cli lighthouse
```

**French (`LANG: fr`):**
```
AVERTISSEMENT : Mode dégradé total — ni Playwright ni aucun scanner CLI n'était disponible.
Cet audit n'a pu effectuer aucune vérification automatisée. Pour un audit complet :
1. Configurez le serveur MCP Playwright (voir https://code.claude.com/docs)
2. Installez les scanners d'accessibilité : npm install -g pa11y axe-cli lighthouse
```
