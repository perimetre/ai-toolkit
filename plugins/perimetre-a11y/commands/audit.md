---
description: "Automated accessibility audit: crawls a web property with Playwright, runs pa11y/axe/Lighthouse, maps findings to WCAG 2.2 and Canadian law, and produces a prioritized HIGH/MEDIUM/LOW report."
argument-hint: "<url> [--jurisdiction global|federal|ontario|quebec] [--depth N] [--lang en|fr]"
allowed-tools: [Bash, Task, AskUserQuestion, mcp__playwright__browser_navigate, mcp__playwright__browser_close, mcp__plugin_playwright_playwright__browser_navigate, mcp__plugin_playwright_playwright__browser_close]
---

# Périmètre Accessibility Audit

Run a full automated accessibility audit on a web property.

## Arguments

| Argument | Default | Description |
|----------|---------|-------------|
| `<url>` | _(required)_ | The seed URL to audit (must be http:// or https://) |
| `--jurisdiction` | `global` | Legal jurisdiction: `global`, `federal`, `ontario`, or `quebec` |
| `--depth N` | `2` | BFS crawl depth (1 = seed URL only, 2 = seed + direct links, …, max 20) |
| `--lang` | `en` | Report language: `en` (English) or `fr` (French) |

---

## Step 0: Parse and Validate Arguments

### Mode A — URL provided in arguments

If a positional URL was passed (e.g. `/perimetre-a11y:audit https://example.com`), extract it and all flags. Use these defaults for any omitted flag: jurisdiction = `global`, depth = `2`, lang = `en`. Do not ask any questions — proceed directly to Step 1.

1. **URL** — first non-flag argument. Must start with `http://` or `https://`. If invalid, stop and report the error.
2. **`--jurisdiction`** — valid values: `global`, `federal`, `ontario`, `quebec`. Default: `global`.
3. **`--depth N`** — integer, clamped 1–20. Default: `2`.
4. **`--lang`** — valid values: `en`, `fr`. Default: `en`.

### Mode B — No URL in arguments

If **no positional URL was provided** (command invoked with no arguments, or with flags only), ask all four questions in sequence before doing anything else. Do not infer, prefill, or suggest any values — present each question as a blank slate.

**Question 1 — Report language:**
```
AskUserQuestion(
  question: "In which language should the report be written?",
  options: [
    "English",
    "Français"
  ]
)
```
Map `"English"` → `en`, `"Français"` → `fr`.

**Question 2 — Jurisdiction:**
```
AskUserQuestion(
  question: "Which legal jurisdiction applies to this site?",
  options: [
    "Global — WCAG 2.2 only, no legal citations",
    "Federal — ACA 2019, Canada",
    "Ontario — AODA",
    "Quebec — SGQRI / Charte"
  ]
)
```
Map: `"Global…"` → `global`, `"Federal…"` → `federal`, `"Ontario…"` → `ontario`, `"Quebec…"` → `quebec`.

**Question 3 — Crawl depth:**
```
AskUserQuestion(
  question: "How many pages deep should the crawler go?",
  options: [
    "1 — Seed page only",
    "2 — Seed page + direct links (default)",
    "5 — Multi-section crawl",
    "10 — Deep crawl",
    "20 — Maximum depth"
  ]
)
```
Extract the leading number from the selected option. If parsing fails, use `2`.

**Question 4 — URL:**

Use `AskUserQuestion` with this exact question text — do not modify it.

```
AskUserQuestion(
  question: "URL :"
)
```

After receiving the answer, validate silently (must start with `http://` or `https://`). If invalid, ask again the same way.

---

After collecting all values (from either mode), derive the audit date (today's date in ISO format YYYY-MM-DD), then echo the confirmed configuration:

```
Accessibility Audit
===================
URL:          [url]
Jurisdiction: [jurisdiction]
Depth:        [depth]
Language:     [lang]
Date:         [audit_date]

Running pre-flight check...
```

---

## Step 1: Pre-flight Check

Run the following checks **before** launching any agents. This step ensures missing tools are caught immediately rather than discovered mid-audit.

### 1a. Check CLI tools

Run each command and capture whether it succeeds:

```bash
npx pa11y --version 2>/dev/null && echo "PA11Y_FOUND" || echo "PA11Y_MISSING"
npx axe --version 2>/dev/null && echo "AXE_FOUND" || echo "AXE_MISSING"
npx lighthouse --version 2>/dev/null && echo "LH_FOUND" || echo "LH_MISSING"
```

Parse each result:
- `*_FOUND` → record version string if captured, status = available
- `*_MISSING` → status = not found

### 1b. Check Playwright

Attempt `browser_navigate` to `about:blank` using the Playwright MCP tool (either `mcp__playwright__browser_navigate` or `mcp__plugin_playwright_playwright__browser_navigate`). If the tool call succeeds, close the browser with `browser_close` and mark Playwright as available. If the tool is not found or throws an error, mark Playwright as unavailable.

### 1c. Print status table

Print the following table in the chosen language (`LANG`):

**English (`LANG: en`):**
```
Pre-flight Check
================
| Tool        | Status                        |
|-------------|-------------------------------|
| Playwright  | ✅ Available / ❌ Not found   |
| pa11y       | ✅ v3.x / ❌ Not found        |
| axe         | ✅ v4.x / ❌ Not found        |
| Lighthouse  | ✅ v12.x / ❌ Not found       |
```

**French (`LANG: fr`):**
```
Vérification préliminaire
=========================
| Outil       | Statut                        |
|-------------|-------------------------------|
| Playwright  | ✅ Disponible / ❌ Introuvable |
| pa11y       | ✅ v3.x / ❌ Introuvable       |
| axe         | ✅ v4.x / ❌ Introuvable       |
| Lighthouse  | ✅ v12.x / ❌ Introuvable      |
```

### 1d. Handle missing tools

If **any** CLI tool is missing (pa11y, axe, Lighthouse):

1. List each missing tool with its install command
2. If Playwright is also unavailable AND depth > 1: note that the sitemap fallback will be used for URL discovery (depth parameter will have no effect)
3. Use `AskUserQuestion` with the message (in `LANG`):

   **English:** "Some accessibility tools are missing. How would you like to proceed?"
   **French:** "Certains outils d'accessibilité sont manquants. Comment souhaitez-vous procéder ?"

   Options (in `LANG`):
   - **EN:** "Install automatically (npm install -g pa11y axe-cli lighthouse)" / **FR:** "Installer automatiquement (npm install -g pa11y axe-cli lighthouse)"
   - **EN:** "Show me the commands — I'll install manually" / **FR:** "Afficher les commandes — je les installerai moi-même"
   - **EN:** "Continue without these tools (degraded coverage)" / **FR:** "Continuer sans ces outils (couverture réduite)"
   - **EN:** "Cancel" / **FR:** "Annuler"

   **If user chooses auto-install:**
   Run `npm install -g pa11y axe-cli lighthouse` (or individual commands for missing tools only). Re-run the version checks. Print updated status table. Then proceed.

   **If user chooses manual install:**
   Show exact install commands, then **stop** — do not proceed.

   **If user chooses continue:**
   Prepend a degraded coverage warning to the pipeline output and proceed. Store `PREFLIGHT_DEGRADED: true`.

   **If user cancels:**
   Stop entirely — do not proceed.

If **all three** CLI tools are unavailable AND Playwright is also unavailable AND user chose continue: **stop** — no meaningful audit is possible without at least one data source.

### 1e. Proceed

After the pre-flight check passes (or user chose to continue with degraded coverage), print:

**English:** `Starting pipeline...`
**French:** `Démarrage du pipeline...`

---

## Step 2: Crawl

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

Inform the user: `Step 2 complete — [PAGES_CRAWLED] pages crawled (Playwright: [PLAYWRIGHT_STATUS])`

---

## Step 3: Scan

Use the `scanner-runner` agent via Task to run CLI accessibility scanners.

Pass in the prompt:
```
Run accessibility scanners against the following page inventory.

PAGE_INVENTORY:
[PAGE_INVENTORY JSON]

SEED_URL: [url]
JURISDICTION: [jurisdiction]
PLAYWRIGHT_STATUS: [PLAYWRIGHT_STATUS]

Run pa11y, axe, and Lighthouse best-effort for up to 20 URLs.
```

Store the full agent output as `SCANNER_OUTPUT`.

Extract from `SCANNER_OUTPUT`:
- `SCANNER_STATUS` (look for `SCANNER_STATUS:` line)
- `PA11Y_STATUS`, `AXE_STATUS`, `LIGHTHOUSE_STATUS`
- `SCANNER_FINDINGS` JSON array
- `SCANNER_NOTES` text

Inform the user: `Step 3 complete — scanners: pa11y=[PA11Y_STATUS] axe=[AXE_STATUS] lighthouse=[LIGHTHOUSE_STATUS]`

---

## Step 4: Map

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

Inform the user: `Step 4 complete — [TOTAL_ISSUES] issues mapped (HIGH: [HIGH_COUNT], MEDIUM: [MEDIUM_COUNT], LOW: [LOW_COUNT])`

---

## Short-Circuit: No Issues Found

If `TOTAL_ISSUES` is 0, output the no-findings message in the chosen language and stop. Do not launch the report-writer agent.

For the page count: use `PAGES_CRAWLED` from the crawler output if available; if `PLAYWRIGHT_STATUS` is `playwright-unavailable`, use `1` (the seed URL).

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

## Step 5: Report

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
