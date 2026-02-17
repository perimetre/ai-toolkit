---
name: brand-guidelines
description: Load Périmètre's visual identity guidelines into context. Use this skill whenever producing any branded deliverable — documents, presentations, emails, social content, or any other output that represents Périmètre externally or internally. This skill ensures consistent use of colours, typography, and logo across all work.
---

# Périmètre Brand Guidelines

You are now operating with full knowledge of Périmètre's visual identity. Apply these guidelines to every deliverable produced in this session unless the user explicitly asks you not to.

---

## Colour Palette

Périmètre uses a restrained, professional palette built around two neutral primaries and one bold accent.

### Primary Colours

**Carbon** — The dominant colour. Used for body text, headings, backgrounds, and primary UI elements.
- HEX: `#111111`
- RGB: 17, 17, 17
- CMYK: 0, 0, 0, 93
- Usage: Main text colour, dark backgrounds, primary fills. Near-black — conveys precision and solidity.

**Steel** — The secondary neutral. Used for supporting text, borders, dividers, captions, and muted UI elements.
- HEX: `#737678`
- RGB: 115, 118, 120
- CMYK: 4, 2, 0, 53
- Usage: Secondary text, subtle borders, metadata, placeholder text, footnotes. Use to create hierarchy without introducing new colours.

### Accent Colour

*(Name TBC)* — The brand accent. A vivid cyan used for highlights, underlines, active states, decorative borders, and calls to action.
- HEX: `#13FADC`
- RGB: 19, 250, 220
- Usage: Highlights, underlines, accent borders, tags, badges, key data callouts. Never use as a large background fill — it should draw the eye. Overuse will dilute its impact.

### Colour Usage Rules

- **Never use the accent for body text** — it lacks sufficient contrast on white backgrounds.
- **Never use more than one accent colour** — `#13FADC` is the only accent in the palette; do not introduce additional colours.
- **White (`#FFFFFF`)** is always acceptable as a background or reversed text colour on Carbon backgrounds.
- **Dark-on-light is the default** — Carbon text on white or light backgrounds. Light-on-dark (white on Carbon) is appropriate for headers, covers, and hero sections.
- When using colour in charts or data visualisations: use Carbon for primary series, Steel for secondary series, and `#13FADC` sparingly for the most important data point or trendline.

---

## Typography

### Primary Typeface: Satoshi

All Périmètre communications use **Satoshi** as the primary typeface. Satoshi is a modern geometric sans-serif — clean, confident, and highly legible at all sizes.

- **Weights available**: Light (300), Regular (400), Medium (500), Bold (700), Black (900)
- **Fallback stack**: `Satoshi, -apple-system, BlinkMacSystemFont, 'Helvetica Neue', Arial, sans-serif`

### Type Hierarchy

| Level | Weight | Notes |
|---|---|---|
| Display / Hero | Black (900) or Bold (700) | Large headings on covers, hero sections |
| H2 | Bold (700) | Primary section headings |
| H3 | Medium (500) | Sub-section headings, card titles |
| Body | Regular (400) | Main readable text |
| Caption / Label | Regular (400) or Light (300) | Metadata, footnotes, captions |
| Emphasis | Medium (500) | Use instead of italic; Satoshi reads better in medium weight than italic |

### Typography Rules

- **Use sentence case** for headings and labels — avoid ALL CAPS except for very short tags (e.g., "NEW", "BETA").
- **Line height**: 1.5–1.6× for body text; tighter for display headings.
- **Letter spacing**: Negative tracking on headings is intentional and characteristic of the brand.
- **Never use more than two weights in a single layout** — choose a pairing (e.g., Bold + Regular) and stick to it.
- **Avoid decorative or script fonts** — Satoshi is the sole typeface. Do not supplement with other fonts.

---

## Logo Usage

### Core Rules

- **Always use the official logo files** from `assets/logos/` — do not recreate the logo in text or simple shapes.
- **Minimum clear space**: Maintain clear space around the logo equal to the height of the logo mark on all sides.
- **Minimum size**: Do not reproduce the logo smaller than 80px wide in digital contexts or 20mm wide in print.
- **Language matching**: Use the French logotype (`-fr-`) for French-language contexts and the English logotype (`-en-`) for English. The difference is typographic: the French wordmark carries the correct accents (é, è).

### The Mark

The logo mark is a geometric square frame — three concentric, chamfered square outlines — a direct visual reference to "périmètre" (perimeter). It is always rendered as a single flat colour: black on light backgrounds, white on dark.

### Available Variants

All files are in `assets/logos/`.

#### Logo mark only — 147×147px
| File | Fill | Use on |
|---|---|---|
| `perimetre-logo-dark.svg` | Black | White or light backgrounds |
| `perimetre-logo-light.svg` | White | Dark (`#111111`) backgrounds |

#### English logotype, horizontal — 458×147px
| File | Fill | Use on |
|---|---|---|
| `perimetre-logotype-english-horizontal-dark.svg` | Black | White or light backgrounds |
| `perimetre-logotype-english-horizontal-light.svg` | White | Dark (`#111111`) backgrounds |

Wordmark: **perimeter** (no accents — intentional for the English variant).

#### French logotype, horizontal — 457×147px
| File | Fill | Use on |
|---|---|---|
| `perimetre-logotype-french-horizontal-dark.svg` | Black | White or light backgrounds |
| `perimetre-logotype-french-horizontal-light.svg` | White | Dark (`#111111`) backgrounds |

Wordmark: **périmètre** (with accents é and è).

#### French logotype, vertical — 147×119px
| File | Fill | Use on |
|---|---|---|
| `perimetre-logotype-french-vertical-dark.svg` | Black | White or light backgrounds |

Mark sits above the wordmark. Use when horizontal space is constrained (e.g. square social avatars, mobile headers).

### Choosing the Right Variant

- **Default**: horizontal logotype in the language of the document
- **Tight or square spaces** (favicons, avatars, app icons, stamps): logo mark only
- **Dark backgrounds** (covers, hero sections, footers): use the `-light` variant
- **Light backgrounds** (body pages, emails, most slides): use the `-dark` variant

### What to Avoid

- Do not recreate or approximate the logo in text or basic shapes.
- Do not place the logo on complex photographic backgrounds without a clean container or overlay.
- Do not recolour the logo — only black and white variants are approved.
- Do not stretch, skew, rotate, or apply effects (shadows, outlines, gradients) to the logo.
- Do not use the English logotype in French-language documents, or vice versa.

---

## Applying the Brand in Deliverables

### Documents (Word, PDF)

- Cover: Carbon background, white text, `#13FADC` accent border or rule line.
- Body pages: White background, Carbon text, Steel for secondary information.
- Use `#13FADC` sparingly: section tags, key callout boxes, important data points.
- Table headers: Carbon background, white text, or Steel fill with Carbon text.

### Presentations (PowerPoint, Google Slides)

- Title slide: Carbon background, white headline, `#13FADC` accent element.
- Content slides: White background with Carbon text as default.
- Use a thin `#13FADC` rule or accent bar to add brand identity to slide headers.
- Charts: Carbon and Steel for most series; `#13FADC` to highlight the key insight.

### Emails and Written Communications

- Plain text environments: Brand colours are not directly applicable; focus on tone.
- HTML emails: Use Carbon for headers and body text, `#13FADC` for CTA buttons or accent borders, Steel for dividers and footer text.

### Tone of Voice (General Guidance)

Périmètre's written voice reflects its visual identity — precise, confident, and uncluttered.
- **Concise**: Say more with less. No filler phrases, no jargon for its own sake.
- **Direct**: Lead with the point. Don't bury conclusions.
- **Professional but not stiff**: Warm and human, not corporate or cold.
- **French/English**: Périmètre operates bilingually. Match the language of the recipient; default to the language of the user's request.

---

## Quick Reference

| Element | Value |
|---|---|
| Primary text colour | `#111111` (Carbon) |
| Secondary / muted text | `#737678` (Steel) |
| Accent / highlight | `#13FADC` *(name TBC)* |
| Background (default) | `#FFFFFF` (White) |
| Background (dark) | `#111111` (Carbon) |
| Primary font | Satoshi |
| Fallback font stack | Satoshi, Helvetica Neue, Arial, sans-serif |
| Heading weight | Bold (700) |
| Body weight | Regular (400) |
