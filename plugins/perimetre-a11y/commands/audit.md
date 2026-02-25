---
description: "Automated accessibility audit: crawls a web property with agent-browser, runs pa11y/axe/Lighthouse, maps findings to WCAG 2.2 and Canadian law, and produces a prioritized HIGH/MEDIUM/LOW report."
argument-hint: "<url> [--jurisdiction global|federal|ontario|quebec] [--depth N] [--lang en|fr]"
allowed-tools: [Bash, Task, AskUserQuestion]
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

If a positional URL was passed (e.g. `/perimetre-a11y:audit https://your-site.com`), extract it and all flags. Use these defaults for any omitted flag: jurisdiction = `global`, depth = `2`, lang = `en`. Do not ask any questions — proceed directly to Step 1.

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

If `lang` is `en`, execute this call EXACTLY as written:
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

If `lang` is `fr`, execute this call EXACTLY as written:
```
AskUserQuestion(
  question: "Quelle profondeur d'exploration souhaitez-vous ?",
  options: [
    "1 — Page de départ seulement",
    "2 — Page de départ + liens directs (défaut)",
    "5 — Exploration multi-sections",
    "10 — Exploration approfondie",
    "20 — Profondeur maximale"
  ]
)
```

Extract the leading number from the selected option. If parsing fails, use `2`.

**Question 4 — URL:**

The URL is the user's own specific website — it is impossible to know in advance. Options for a URL field make no sense. Execute this call EXACTLY as written, with no additions:

```
AskUserQuestion(
  question: "URL du site / Site URL :"
)
```

After receiving the answer, validate silently. If the answer is not a valid web address, call the same question again.

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

### 1b. Check agent-browser

```bash
agent-browser open about:blank 2>/dev/null && agent-browser close && echo "AGENT_BROWSER_FOUND" || echo "AGENT_BROWSER_MISSING"
```

If the output is `AGENT_BROWSER_FOUND`, mark agent-browser as available. If `AGENT_BROWSER_MISSING`, mark agent-browser as unavailable.

### 1c. Print status table

Print the following table in the chosen language (`LANG`):

**English (`LANG: en`):**
```
Pre-flight Check
================
| Tool          | Status                        |
|---------------|-------------------------------|
| agent-browser | ✅ Available / ❌ Not found   |
| pa11y         | ✅ v3.x / ❌ Not found        |
| axe           | ✅ v4.x / ❌ Not found        |
| Lighthouse    | ✅ v12.x / ❌ Not found       |
```

**French (`LANG: fr`):**
```
Vérification préliminaire
=========================
| Outil         | Statut                        |
|---------------|-------------------------------|
| agent-browser | ✅ Disponible / ❌ Introuvable |
| pa11y         | ✅ v3.x / ❌ Introuvable       |
| axe           | ✅ v4.x / ❌ Introuvable       |
| Lighthouse    | ✅ v12.x / ❌ Introuvable      |
```

### 1d. Handle missing tools

If **any** tool is missing (pa11y, axe, Lighthouse, or agent-browser):

1. List **every** missing tool with its install command:
   - pa11y: `npm install -g pa11y`
   - axe: `npm install -g axe-cli`
   - Lighthouse: `npm install -g lighthouse`
   - agent-browser: `npm install -g @vercel/agent-browser`
2. If agent-browser is unavailable AND depth > 1: note that the sitemap fallback will be used for URL discovery (depth parameter will have no effect)
3. Build the auto-install command from **only the missing tools**. Examples:
   - pa11y + axe missing → `npm install -g pa11y axe-cli`
   - axe + agent-browser missing → `npm install -g axe-cli @vercel/agent-browser`
   - all four missing → `npm install -g pa11y axe-cli lighthouse @vercel/agent-browser`
4. Use `AskUserQuestion` with the message (in `LANG`):

   **English:** "Some accessibility tools are missing. How would you like to proceed?"
   **French:** "Certains outils d'accessibilité sont manquants. Comment souhaitez-vous procéder ?"

   Options (in `LANG`) — substitute `[INSTALL_CMD]` with the command built in step 3:
   - **EN:** "Install automatically ([INSTALL_CMD])" / **FR:** "Installer automatiquement ([INSTALL_CMD])"
   - **EN:** "I'll install manually — show me the commands" / **FR:** "Je vais installer manuellement — afficher les commandes"
   - **EN:** "Continue without these tools (degraded coverage)" / **FR:** "Continuer sans ces outils (couverture réduite)"
   - **EN:** "Cancel" / **FR:** "Annuler"

   **If user chooses auto-install:**
   Run the `[INSTALL_CMD]` command built in step 3. Re-run the version checks. Print updated status table.
   If any tools are **still missing** after the install attempt, restart step 1d for those remaining tools — do not silently proceed. The user must explicitly choose how to handle each unresolved tool.

   **If user chooses manual install:**
   Show the exact install commands for each missing tool. Then ask again (in `LANG`):
   - **EN:** "Ready to continue?" / **FR:** "Prêt à continuer ?"
   - Options — **EN:** "Yes, continue" / "Cancel" · **FR:** "Oui, continuer" / "Annuler"
   If user confirms, proceed. If user cancels, stop entirely.

   **If user chooses continue:**
   Prepend a degraded coverage warning to the pipeline output and proceed. Store `PREFLIGHT_DEGRADED: true`.

   **If user cancels:**
   Stop entirely — do not proceed.

If **all three** CLI tools are unavailable AND agent-browser is also unavailable AND user chose continue: **stop** — no meaningful audit is possible without at least one data source.

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
- `BROWSER_STATUS` (look for `BROWSER_STATUS:` line)
- `PAGE_INVENTORY` JSON array (content between `PAGE INVENTORY` and `DOM FINDINGS` sections)
- `DOM_FINDINGS` JSON array (content between `DOM FINDINGS` and `CRAWL NOTES` sections)
- `PAGES_CRAWLED` count
- `CRAWL_NOTES` text

Inform the user: `Step 2 complete — [PAGES_CRAWLED] pages crawled (agent-browser: [BROWSER_STATUS])`

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
BROWSER_STATUS: [BROWSER_STATUS]

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

BROWSER_STATUS: [BROWSER_STATUS]
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

For the page count: use `PAGES_CRAWLED` from the crawler output if available; if `BROWSER_STATUS` is `browser-unavailable`, use `1` (the seed URL).

**English (`LANG: en`):**
```
No accessibility issues detected across [PAGES_CRAWLED] page(s).

Scanners: [list available scanners]
Crawl depth: [depth]
Jurisdiction: [jurisdiction]

[If BROWSER_STATUS is browser-unavailable, add:]
Note: DOM-level checks were not performed (agent-browser unavailable). Install agent-browser
(`npm i -g @vercel/agent-browser` or see https://agent-browser.dev) for more thorough analysis.

[If SCANNER_STATUS is no-scanners, add:]
Note: No CLI scanners were available. Install pa11y, axe-cli, and lighthouse for scanner coverage.
```

**French (`LANG: fr`):**
```
Aucun problème d'accessibilité détecté sur [PAGES_CRAWLED] page(s).

Scanners : [list available scanners]
Profondeur d'exploration : [depth]
Juridiction : [jurisdiction]

[If BROWSER_STATUS is browser-unavailable, add:]
Note : Les vérifications DOM n'ont pas pu être effectuées (agent-browser non disponible). Installez
agent-browser (`npm i -g @vercel/agent-browser` ou voir https://agent-browser.dev) pour une analyse
DOM complète.

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

BROWSER_STATUS: [BROWSER_STATUS]
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

If both `BROWSER_STATUS` is `browser-unavailable` AND `SCANNER_STATUS` is `no-scanners`, prepend the following before the report in the chosen language:

**English (`LANG: en`):**
```
WARNING: Full degraded mode — neither agent-browser nor any CLI scanner was available.
This audit could not perform any automated checks. To run a full audit:
1. Install agent-browser: npm i -g @vercel/agent-browser (see https://agent-browser.dev)
2. Install accessibility scanners: npm install -g pa11y axe-cli lighthouse
```

**French (`LANG: fr`):**
```
AVERTISSEMENT : Mode dégradé total — ni agent-browser ni aucun scanner CLI n'était disponible.
Cet audit n'a pu effectuer aucune vérification automatisée. Pour un audit complet :
1. Installez agent-browser : npm i -g @vercel/agent-browser (voir https://agent-browser.dev)
2. Installez les scanners d'accessibilité : npm install -g pa11y axe-cli lighthouse
```
