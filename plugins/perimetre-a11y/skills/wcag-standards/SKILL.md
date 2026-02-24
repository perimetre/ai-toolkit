---
name: wcag-standards
description: >
  Technical accessibility reference for WCAG 2.2 auditing. Covers POUR principles, Level A and AA
  success criteria, common failure patterns, DOM inspection checklist (13 checks), and the
  unified scoring formula used by the perimetre-a11y pipeline.
tools: []
skills: []
---

# WCAG 2.2 Standards Reference

## POUR Principles

| Principle | Meaning |
|-----------|---------|
| **Perceivable** | Information and UI components must be presentable to users in ways they can perceive (not invisible to all senses) |
| **Operable** | UI components and navigation must be operable — all functionality available from keyboard, sufficient time, no seizure risk |
| **Understandable** | Information and UI operation must be understandable — readable, predictable, input assistance |
| **Robust** | Content must be robust enough to be reliably interpreted by assistive technologies — valid markup, name/role/value |

---

## Level A Success Criteria

| SC | Short Name | What to Check |
|----|-----------|---------------|
| 1.1.1 | Non-text Content | All `<img>`, `<area>`, `<input type="image">` have meaningful `alt`; decorative images have `alt=""` or `role="presentation"` |
| 1.2.1 | Audio-only / Video-only (pre-recorded) | Pre-recorded audio-only has transcript; pre-recorded video-only has audio track or transcript |
| 1.2.2 | Captions (pre-recorded) | Pre-recorded video with audio has synchronized captions |
| 1.2.3 | Audio Description or Media Alternative | Pre-recorded video has audio description or full text alternative |
| 1.3.1 | Info and Relationships | Structure conveyed visually (headings, lists, tables) is programmatically determinable via HTML semantics or ARIA |
| 1.3.2 | Meaningful Sequence | Reading order in DOM matches visual reading order when sequence affects meaning |
| 1.3.3 | Sensory Characteristics | Instructions don't rely solely on shape, size, visual location, orientation, or sound |
| 1.4.1 | Use of Color | Color is not the only visual means of conveying information, prompting action, or distinguishing elements |
| 2.1.1 | Keyboard | All functionality operable via keyboard; no keyboard traps |
| 2.1.2 | No Keyboard Trap | Keyboard focus can always be moved away from a component using only the keyboard |
| 2.2.1 | Timing Adjustable | Time limits can be turned off, adjusted, or extended (20× minimum) |
| 2.2.2 | Pause, Stop, Hide | Moving/auto-updating content can be paused, stopped, or hidden |
| 2.3.1 | Three Flashes or Below Threshold | No content flashes more than 3× per second (or below general/red flash thresholds) |
| 2.4.1 | Bypass Blocks | Mechanism to skip repeated navigation (skip link, landmark, heading) |
| 2.4.2 | Page Titled | Each page has a descriptive `<title>` element |
| 2.4.3 | Focus Order | Focus order preserves meaning and operability |
| 2.4.4 | Link Purpose (In Context) | Link purpose determinable from link text alone or link text + context |
| 2.5.3 | Label in Name | Visible label text is contained in the accessible name of interactive elements |
| 2.5.4 | Motion Actuation | Functions triggered by device motion have UI alternatives; can be disabled |
| 3.1.1 | Language of Page | `<html lang="...">` present and valid |
| 3.2.1 | On Focus | Focus doesn't trigger unexpected context change |
| 3.2.2 | On Input | Input doesn't trigger unexpected context change without prior notice |
| 3.3.1 | Error Identification | Input errors identified in text and described |
| 3.3.2 | Labels or Instructions | Labels or instructions present when content requires user input |
| 4.1.1 | Parsing | No duplicate IDs, unclosed tags, incorrect nesting |
| 4.1.2 | Name, Role, Value | All UI components have accessible name, role, and state/value determinable |
| 4.1.3 | Status Messages | Status messages programmatically determinable via role or property (no focus needed) |

---

## Level AA Success Criteria

| SC | Short Name | What to Check |
|----|-----------|---------------|
| 1.2.4 | Captions (Live) | Live video with audio has real-time captions |
| 1.2.5 | Audio Description (pre-recorded) | Pre-recorded video has audio description |
| 1.3.4 | Orientation | Content doesn't restrict to a single display orientation unless essential |
| 1.3.5 | Identify Input Purpose | `autocomplete` attributes match WCAG-defined tokens on personal data inputs |
| 1.4.3 | Contrast (Minimum) | Text/images of text have 4.5:1 contrast ratio (3:1 for large text ≥18pt or 14pt bold) |
| 1.4.4 | Resize Text | Text can be resized 200% without loss of content or functionality |
| 1.4.5 | Images of Text | Text used instead of images of text (except logos, essential images) |
| 1.4.10 | Reflow | Content reflows at 320px width without horizontal scrolling (except 2D content) |
| 1.4.11 | Non-text Contrast | UI components and graphical objects have 3:1 contrast against adjacent colors |
| 1.4.12 | Text Spacing | No loss of content when overriding line-height ≥1.5×, letter-spacing ≥0.12em, word-spacing ≥0.16em |
| 1.4.13 | Content on Hover or Focus | Content appearing on hover/focus is dismissible, hoverable, and persistent |
| 2.4.5 | Multiple Ways | More than one way to locate a page (site map, search, TOC) |
| 2.4.6 | Headings and Labels | Headings and labels describe topic or purpose |
| 2.4.7 | Focus Visible | Keyboard focus indicator visible |
| 2.4.11 | Focus Not Obscured (Minimum) | Component receiving focus not entirely hidden by sticky/fixed content |
| 2.5.7 | Dragging Movements | Single-pointer alternative for all dragging operations |
| 2.5.8 | Target Size (Minimum) | Target size ≥24×24 CSS px (or adequate spacing/inline exemption) |
| 3.1.2 | Language of Parts | Language changes within page identified with `lang` attribute |
| 3.2.3 | Consistent Navigation | Repeated navigation appears in same relative order |
| 3.2.4 | Consistent Identification | Components with same functionality identified consistently |
| 3.3.3 | Error Suggestion | Input errors include suggestions for correction when known |
| 3.3.4 | Error Prevention (Legal, Financial, Data) | Submissions reversible, checked, or confirmed |

---

## Common Failure Patterns → SC Mapping

| Failure Pattern | Primary SC | Secondary SC |
|-----------------|-----------|--------------|
| `<img>` missing `alt` | 1.1.1 | 4.1.2 |
| `<img>` with `alt` same as filename | 1.1.1 | — |
| `<input>` with no `<label>` or `aria-label` | 1.3.1 | 4.1.2 |
| `<button>` with no text / icon-only, no `aria-label` | 4.1.2 | 2.4.4 |
| `<a>` with "click here" or URL as text | 2.4.4 | — |
| `<a>` with no content | 4.1.2 | 2.4.4 |
| Heading levels skipped (h1 → h3) | 1.3.1 | 2.4.6 |
| Missing `<main>`, `<nav>`, `<header>`, `<footer>` | 1.3.1 | 2.4.1 |
| `tabindex` > 0 | 2.4.3 | — |
| `outline: none` / `outline: 0` suppressing focus ring | 2.4.7 | 2.4.11 |
| Interactive element < 24×24 px | 2.5.8 | — |
| Low contrast text | 1.4.3 | — |
| Low contrast UI component | 1.4.11 | — |
| `<html>` missing or invalid `lang` | 3.1.1 | — |
| `<title>` empty or missing | 2.4.2 | — |
| `<iframe>` missing `title` | 4.1.2 | — |
| `<table>` missing `<caption>` or `<th scope>` | 1.3.1 | — |
| `role="dialog"` missing `aria-labelledby` | 4.1.2 | — |
| ARIA `required` state not reflected on control | 4.1.2 | 3.3.1 |
| Color alone distinguishes form error | 1.4.1 | 3.3.1 |
| Duplicate `id` attribute values | 4.1.1 | — |
| Visible label text not in accessible name | 2.5.3 | — |

---

## DOM Inspection Checklist (13 Checks)

Run these per page via Playwright `evaluate()` or snapshot analysis:

| # | Check | Pass Condition |
|---|-------|---------------|
| 1 | **Missing alt text** | `document.querySelectorAll('img:not([alt])')` returns 0 elements |
| 2 | **Unlabeled form controls** | Every `input:not([type=hidden],[type=submit],[type=reset],[type=button])`, `select`, `textarea` has an associated `<label>` or `aria-label` or `aria-labelledby` |
| 3 | **Heading sequence gaps** | Heading levels (h1–h6) in DOM order never skip more than one level |
| 4 | **Missing landmarks** | Page contains at least one `<main>` or `role="main"`; has `<nav>` or `role="navigation"` if there is repeated navigation |
| 5 | **Links/buttons with no accessible name** | `document.querySelectorAll('a,button')` — none have empty text content AND no `aria-label` AND no `aria-labelledby` AND no `title` |
| 6 | **Small interactive targets** | Interactive elements (a, button, [role=button], input, select, textarea) with rendered width or height < 24px AND no adequate spacing |
| 7 | **Suppressed focus indicators** | CSS does not contain `outline: none` or `outline: 0` on `:focus` or `:focus-visible` without a replacement focus style |
| 8 | **tabindex > 0** | No element has `tabindex` attribute value greater than 0 |
| 9 | **Missing lang attribute** | `document.documentElement.lang` is non-empty and a valid BCP-47 tag |
| 10 | **Missing or empty title** | `document.title` is non-empty and descriptive (not just the domain) |
| 11 | **iframe without title** | `document.querySelectorAll('iframe:not([title])')` returns 0 elements |
| 12 | **Duplicate IDs** | No `id` attribute value appears more than once in the DOM |
| 13 | **Images of text** | `<img>` elements whose `alt` text matches the image content are flagged for manual review (automated signal only) |

---

## Scoring Formula

### Base Scores by Source

| Source | Impact Level | Base Score |
|--------|-------------|------------|
| axe-core | critical | 90 |
| axe-core | serious | 80 |
| axe-core | moderate | 65 |
| axe-core | minor | 45 |
| pa11y | error | 72 |
| pa11y | warning | 56 |
| pa11y | notice | 40 |
| Lighthouse | failed audit (accessibility) | 52–70 (scaled by impact weight) |
| DOM structural (headings, landmarks, lang, title) | — | 50–75 |
| DOM interactive (missing labels, names, focus) | — | 60–75 |
| DOM visual (target size, focus indicator) | — | 50–65 |

### Boost Modifiers (additive, cap at 100)

| Condition | Boost |
|-----------|-------|
| SC is contrast (1.4.3), focus (2.4.7), label (1.3.1/4.1.2), or target size (2.5.8) | +10 |
| Issue has a strong jurisdiction signal (see `canadian-accessibility-law` skill) | +8 |
| Issue found on 3 or more pages | +4 |

### Priority Buckets

| Bucket | Score Range |
|--------|------------|
| **HIGH** | ≥ 85 |
| **MEDIUM** | 60 – 84 |
| **LOW** | < 60 |

### Deduplication Rule

When the same element (or same SC on the same element) is flagged by multiple scanners:
1. Keep the highest base score from all sources
2. Apply boosts once (not per source)
3. Merge `pages` arrays — record all pages where the issue appears
4. Set `source` to `"multi"` and list all contributing sources
