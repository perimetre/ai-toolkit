# perimetre-a11y

**Version:** 1.4.16 — Last updated: 2026-02-25

Automated accessibility auditing for web properties. Crawls with agent-browser, runs pa11y/axe-core/Lighthouse, maps findings to WCAG 2.2 and Canadian jurisdiction law (Federal/Ontario/Quebec), scores each issue 0–100, and produces a lean prioritized report. Gracefully degrades when agent-browser or scanners are unavailable.

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

If invoked without a URL, all options are collected interactively with clear preset choices.

**Examples:**

```bash
# Interactive mode — asks URL, language, jurisdiction, and depth
/perimetre-a11y:audit

# Basic audit (WCAG 2.2, no jurisdiction citations, depth 2)
/perimetre-a11y:audit https://example.com

# Audit with Ontario AODA citations
/perimetre-a11y:audit https://example.com --jurisdiction ontario

# Audit with Federal ACA citations, deeper crawl
/perimetre-a11y:audit https://example.com --jurisdiction federal --depth 10

# Quebec government scope, French report
/perimetre-a11y:audit https://example.gouv.qc.ca --jurisdiction quebec --lang fr

# Single-page audit (seed URL only)
/perimetre-a11y:audit https://example.com --depth 1
```

**Depth guide:**

| `--depth` | Pages scanned |
|-----------|--------------|
| `1` | Seed URL only |
| `2` | Seed + all direct links on the same domain _(default)_ |
| `5` | Multi-section crawl |
| `10` | Deep crawl |
| `20` | Maximum |

---

## Pipeline

The audit runs a 5-stage sequential pipeline:

| Stage | What it does |
|-------|-------------|
| 0 | Argument parsing + interactive collection |
| 1 | **Pre-flight check** — verifies pa11y, axe, Lighthouse, and agent-browser availability before any agents run; offers auto-install if tools are missing |
| 2 | `web-crawler` — BFS crawl up to 50 pages; runs 13 DOM checks per page via agent-browser |
| 3 | `scanner-runner` — runs pa11y, axe-core, and Lighthouse against up to 20 pages; falls back to sitemap discovery when agent-browser is unavailable |
| 4 | `standards-mapper` — normalizes findings, maps to WCAG SCs, deduplicates, scores 0–100, adds law citations |
| 5 | `report-writer` — produces structured Markdown report and saves it to `./a11y-[hostname]-[date].md` |

---

## Report Structure

1. **Header** — site, date, jurisdiction, standard, scope, scanners used
2. **Summary table** — HIGH/MEDIUM/LOW counts at a glance
3. **Audited pages** — table of all URLs visited with page titles and status
4. **Issue legend** — explains A-xxx identifiers and 🔴🟠🟡 priority labels
5. **HIGH findings** — full card per issue (WCAG criterion, problem, element, affected pages, legal obligation, fix)
6. **MEDIUM findings** — same full card format
7. **LOW findings** — full cards (≤10) or compact table grouped by SC (>10)
8. **Compliance table** — jurisdiction-aware title; each tested SC with pass/fail, issue count, and A-xxx cross-references
9. **Manual verification gaps** — WCAG criteria outside automation scope
10. **Audit notes** — crawl warnings, scanner gaps, sitemap fallback notes, degradation notices

Report is saved to `./a11y-[hostname]-[YYYY-MM-DD].md`.

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
| `federal` | ACA, S.C. 2019, c. 10 + CAN/ASC-EN 301 549:2024 | 2.1 AA |
| `ontario` | AODA 2005 + IASR O.Reg.191/11 s.14 | 2.0 AA |
| `quebec` | SGQRI 008-02 v2.1 (gov't) / Charte c.C-12 s.10 (private) | 2.0 AA (gov't) |

---

## Graceful Degradation

The audit continues producing value even when tools are unavailable:

| Situation | Behaviour |
|-----------|-----------|
| Missing CLI tools detected at start | Pre-flight check flags them immediately and offers to auto-install |
| agent-browser not installed | Skips DOM checks; scanner-runner falls back to sitemap discovery for URL list |
| Sitemap found when agent-browser unavailable | Scans up to 20 URLs from sitemap; notes depth parameter had no effect |
| No sitemap found when agent-browser unavailable | Scans seed URL only; notes depth parameter had no effect |
| All scanners absent (`npx` not found) | Skips scanner stage; runs DOM-only audit with warning |
| Both unavailable | Pre-flight stops execution (no meaningful audit possible) |
| Individual scanner missing | Logs the scanner as unavailable; others continue |
| Page navigation timeout | Logs error; continues crawling remaining pages |

---

## Skills

- `wcag-standards` — POUR principles, Level A/AA SC reference tables, common failure patterns, DOM inspection checklist (13 checks), scoring formula
- `canadian-accessibility-law` — Jurisdiction comparison table, Federal/Ontario/Quebec law summaries, SC-level legal signal lookup, citation formats, multi-jurisdiction guidance

---

## Agents

- `web-crawler` — agent-browser BFS crawler with 13 DOM checks
- `scanner-runner` — pa11y / axe-core / Lighthouse runner
- `standards-mapper` — Normalization, deduplication, scoring, law citations
- `report-writer` — Structured Markdown report generator

---

## Prerequisites

For full coverage, install accessibility scanners globally:

```bash
npm install -g pa11y axe-cli lighthouse
```

And install agent-browser for DOM crawling:

```bash
npm install -g agent-browser
```

See the [agent-browser documentation](https://github.com/vercel-labs/agent-browser) for more details.
