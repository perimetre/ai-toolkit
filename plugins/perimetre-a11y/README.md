# perimetre-a11y

**Version:** 1.2.0 — Last updated: 2026-02-25

Automated accessibility auditing for web properties. Crawls with Playwright, runs pa11y/axe-core/Lighthouse, maps findings to WCAG 2.2 and Canadian jurisdiction law (Federal/Ontario/Quebec), scores each issue 0–100, and produces a lean prioritized report. Gracefully degrades when Playwright or scanners are unavailable.

---

## Installation

```bash
/plugin install perimetre-a11y
```

Or install the full marketplace:

```bash
/plugin marketplace add https://github.com/perimetre/ai-toolkit
```

---

## Usage

```
/perimetre-a11y:audit <url> [--jurisdiction global|federal|ontario|quebec] [--depth N] [--lang en|fr]
```

**Examples:**

```bash
# Basic audit (WCAG 2.2, no jurisdiction citations)
/perimetre-a11y:audit https://example.com

# Audit with Ontario AODA citations
/perimetre-a11y:audit https://example.com --jurisdiction ontario

# Audit with Federal ACA citations, deeper crawl
/perimetre-a11y:audit https://example.com --jurisdiction federal --depth 15

# Quebec government scope, French report
/perimetre-a11y:audit https://example.gouv.qc.ca --jurisdiction quebec --lang fr

# French report, global scope
/perimetre-a11y:audit https://example.com --lang fr
```

---

## Pipeline

The audit runs a 4-stage sequential pipeline:

| Stage | Agent | What it does |
|-------|-------|-------------|
| 1 | `web-crawler` | BFS crawl up to 50 pages; runs 13 DOM checks per page via Playwright |
| 2 | `scanner-runner` | Runs pa11y, axe-core, and Lighthouse against up to 20 pages |
| 3 | `standards-mapper` | Normalizes findings, maps to WCAG SCs, deduplicates, scores 0–100, adds law citations |
| 4 | `report-writer` | Produces structured Markdown report with HIGH/MEDIUM/LOW sections |

---

## Report Structure

1. **Header** — site, date, jurisdiction, standard, scope, scanners used
2. **Summary table** — HIGH/MEDIUM/LOW counts at a glance
3. **HIGH findings** — full detail per issue (SC, legal citation, pages affected, element, fix)
4. **MEDIUM findings** — condensed detail
5. **LOW findings** — condensed or grouped by SC if > 10 total
6. **Compliance lens table** — each tested SC with pass/fail and jurisdiction signal
7. **Manual verification gaps** — WCAG criteria outside automation scope
8. **Audit notes** — crawl warnings, scanner gaps, degradation notices

---

## Scoring Formula

Each issue receives a final score 0–100:

| Source | Base score range |
|--------|----------------|
| axe critical | 90 |
| axe serious | 80 |
| axe moderate | 65 |
| axe minor | 45 |
| pa11y error | 72 |
| pa11y warning | 56 |
| pa11y notice | 40 |
| Lighthouse failed | 52–70 |
| DOM structural | 50–75 |
| DOM interactive | 60–75 |

**Boosts:** +10 for high-priority SCs (contrast, focus, labels, target size), +8 for jurisdiction-mandatory SCs, +4 for issues found on 3+ pages.

**Priority buckets:** HIGH ≥ 85 · MEDIUM 60–84 · LOW < 60

---

## Jurisdiction Support

| Jurisdiction | Law | WCAG Version |
|---|---|---|
| `global` | WCAG 2.2 only | 2.2 AA |
| `federal` | ACA S.C. 2019 c.10 + CAN/ASC-EN 301 549:2024 | 2.1 AA |
| `ontario` | AODA 2005 + IASR O.Reg.191/11 s.14 | 2.0 AA |
| `quebec` | SGQRI 008-02 v2.1 (gov't) / Charte c.C-12 s.10 (private) | 2.0 AA (gov't) |

---

## Graceful Degradation

The audit continues producing value even when tools are unavailable:

| Situation | Behaviour |
|-----------|-----------|
| Playwright MCP not configured | Skips DOM checks; runs scanner-only audit with warning |
| All scanners absent (`npx` not found) | Skips scanner stage; runs DOM-only audit with warning |
| Both unavailable | Reports full degradation and stops |
| Individual scanner missing | Logs the scanner as unavailable; others continue |
| Page navigation timeout | Logs error; continues crawling remaining pages |

---

## Skills

- `wcag-standards` — POUR principles, Level A/AA SC reference tables, common failure patterns, DOM inspection checklist (13 checks), scoring formula
- `canadian-accessibility-law` — Jurisdiction comparison table, Federal/Ontario/Quebec law summaries, SC-level legal signal lookup, citation formats, multi-jurisdiction guidance

---

## Agents

- `web-crawler` — Playwright BFS crawler with 13 DOM checks
- `scanner-runner` — pa11y / axe-core / Lighthouse runner
- `standards-mapper` — Normalization, deduplication, scoring, law citations
- `report-writer` — Structured Markdown report generator

---

## Prerequisites

For full coverage, install accessibility scanners globally:

```bash
npm install -g pa11y axe-cli lighthouse
```

And configure the Playwright MCP server in Claude Code (see the [Playwright MCP documentation](https://code.claude.com/docs)).
