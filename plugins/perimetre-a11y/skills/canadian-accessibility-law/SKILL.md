---
name: canadian-accessibility-law
description: >
  Legal reference for Canadian accessibility obligations across four jurisdictions — Federal,
  Ontario, Quebec (government), and Quebec (private). Includes a jurisdiction comparison table,
  WCAG version mandated per jurisdiction, SC-level legal signal lookup, citation formats, and
  multi-jurisdiction guidance.
tools: []
skills: []
---

# Canadian Accessibility Law Reference

## Jurisdiction Comparison Table

| Dimension | Federal | Ontario | Quebec (Gov't) | Quebec (Private) |
|-----------|---------|---------|----------------|-----------------|
| **Primary Law** | Accessible Canada Act (ACA) S.C. 2019, c. 10 | Accessibility for Ontarians with Disabilities Act (AODA) S.O. 2005, c. 11 | Act to ensure compliance with RRSSS requirements | Charter of Human Rights and Freedoms CQLR c. C-12 |
| **Regulation / Standard** | CAN/ASC-EN 301 549:2024 | IASR O. Reg. 191/11, s. 14 | SGQRI 008-02 v2.1 | No WCAG version mandated |
| **WCAG Version Mandated** | WCAG 2.1 AA | WCAG 2.0 AA | WCAG 2.0 AA | N/A (non-discrimination principle) |
| **Scope** | Federally regulated entities (banks, telecoms, broadcasters, federal crown corps, federal public service) | Public sector + private/non-profit organizations with 50+ employees in Ontario | Quebec government institutions and public bodies | All private enterprises (disability non-discrimination) |
| **Key Exemptions** | Proportionality for small entities, undue hardship | Live captions (AODA s.14(6)); pre-recorded audio descriptions (AODA s.14(7)) | Exemptions for undue burden | Undue hardship defense |
| **Enforcement** | Canadian Accessibility Standards Development Organization (CASDO) / ACA complaints to ACA Commissioner | Accessibility Directorate of Ontario / AODA compliance reports | Conseil du trésor | Quebec Human Rights Commission (CDPDJ) |
| **Complaint / Penalty** | ACA s. 102–117 penalties up to $250,000 | AODA administrative penalties up to $100,000/day | Administrative remedies | CDPDJ tribunal remedies |

---

## Detailed Jurisdiction Summaries

### Federal — Accessible Canada Act (ACA)

- **Citation format:** ACA, S.C. 2019, c. 10 + CAN/ASC-EN 301 549:2024 → WCAG 2.1 AA, SC [x.x.x] [Level]
- **WCAG requirement:** WCAG 2.1 Level AA (via CAN/ASC-EN 301 549:2024 which references WCAG 2.1)
- **Who must comply:** All federally regulated entities — banks, insurance, telecoms, broadcasters, airlines, federal crown corporations, Parliament, and the federal public service
- **Deadline:** Organizations must publish accessibility plans and update them every 3 years; barriers must be progressively eliminated
- **WCAG 2.2 status:** Not yet mandated but EN 301 549:2024 adds WCAG 2.2 SCs 2.4.11, 2.4.12, 2.5.7, 2.5.8 — flag as "best practice" under federal scope
- **Key note:** CAN/ASC-EN 301 549:2024 aligns with European EN 301 549 and covers web, mobile, software, hardware, and documents

### Ontario — AODA / IASR

- **Citation format:** AODA, S.O. 2005, c. 11; IASR O. Reg. 191/11, s. 14([subsection])
- **WCAG requirement:** WCAG 2.0 Level AA
- **Who must comply:**
  - Government of Ontario + all public sector organizations: all WCAG 2.0 AA (with exemptions)
  - Private sector / non-profits with 50+ employees: WCAG 2.0 AA for new/redesigned websites
  - Private sector / non-profits with < 50 employees: WCAG 2.0 A only
- **Named exemptions:**
  - Live captions — IASR O. Reg. 191/11, s. 14(6): live captions exempt for organizations with < 50 employees
  - Pre-recorded audio descriptions — IASR O. Reg. 191/11, s. 14(7): exempt for organizations with < 50 employees
- **WCAG 2.1 / 2.2 SCs:** Technically not mandated under AODA; flag as "best practice"
- **Deadlines:** Large organizations and public sector: compliance required since Jan 1, 2021; small private (50+ employees): Jan 1, 2021 for new websites

### Quebec (Government) — SGQRI 008-02

- **Citation format:** SGQRI 008-02 v2.1, critère [x.x.x]; WCAG 2.0 [SC number]
- **WCAG requirement:** WCAG 2.0 Level AA (with SGQRI 008-02 v2.1 interpretation layer)
- **Who must comply:** Quebec government ministries, agencies, public bodies, and government enterprises (listed in RLRQ c. A-6.01)
- **Key note:** SGQRI 008-02 v2.1 maps directly to WCAG 2.0 with some Quebec-specific guidance on French language accessibility
- **WCAG 2.1 / 2.2 SCs:** Not mandated; flag as "best practice"
- **Enforcement:** Conseil du trésor oversight; accessibility statements required on government websites

### Quebec (Private) — Charter of Human Rights and Freedoms

- **Citation format:** Charte des droits et libertés de la personne, RLRQ c. C-12, art. 10 (disability discrimination); CDPDJ complaint reference
- **WCAG requirement:** None mandated by law
- **Basis:** Charter s. 10 prohibits discrimination based on disability; inaccessible digital services may constitute discrimination, creating a duty to accommodate
- **Duty to accommodate:** Must accommodate up to the point of undue hardship
- **Best practice recommendation:** Apply WCAG 2.0 AA as the de facto standard to avoid discrimination claims; flag WCAG 2.1 AA as recommended
- **Key note:** Quebec private sector has no explicit web accessibility regulation as of the knowledge cutoff; accessibility obligations flow from anti-discrimination law

---

## Citation Format Reference

| Jurisdiction | Issue Citation Format |
|---|---|
| Federal | `ACA s.C. 2019 c.10 + CAN/ASC-EN 301 549:2024 → WCAG 2.1 AA, SC [x.x.x] [Level]` |
| Ontario | `AODA 2005 + IASR O.Reg.191/11 s.14 → WCAG 2.0 AA, SC [x.x.x] [Level]` |
| Quebec (gov't) | `SGQRI 008-02 v2.1 → WCAG 2.0 AA, critère [x.x.x]` |
| Quebec (private) | `Charte c.C-12 s.10 (disability non-discrimination) — no WCAG version mandated; WCAG 2.0 AA recommended` |
| Global (no jurisdiction) | `WCAG 2.2, SC [x.x.x] [Level A/AA]` |

---

## SC-Level Legal Signal Lookup

For each WCAG SC, indicates whether it carries **mandatory** legal weight per jurisdiction or is **best-practice only**.

| SC | Federal | Ontario (50+) | Quebec Gov't | Quebec Private |
|----|---------|--------------|-------------|----------------|
| 1.1.1 Non-text Content (A) | mandatory | mandatory | mandatory | best-practice |
| 1.2.2 Captions pre-rec (A) | mandatory | mandatory | mandatory | best-practice |
| 1.2.4 Captions live (AA) | mandatory | mandatory* | mandatory | best-practice |
| 1.3.1 Info & Relationships (A) | mandatory | mandatory | mandatory | best-practice |
| 1.3.4 Orientation (AA) | mandatory | best-practice | best-practice | best-practice |
| 1.3.5 Input Purpose (AA) | mandatory | best-practice | best-practice | best-practice |
| 1.4.1 Use of Color (A) | mandatory | mandatory | mandatory | best-practice |
| 1.4.3 Contrast Minimum (AA) | mandatory | mandatory | mandatory | best-practice |
| 1.4.10 Reflow (AA) | mandatory | best-practice | best-practice | best-practice |
| 1.4.11 Non-text Contrast (AA) | mandatory | best-practice | best-practice | best-practice |
| 1.4.12 Text Spacing (AA) | mandatory | best-practice | best-practice | best-practice |
| 1.4.13 Content on Hover/Focus (AA) | mandatory | best-practice | best-practice | best-practice |
| 2.1.1 Keyboard (A) | mandatory | mandatory | mandatory | best-practice |
| 2.4.2 Page Titled (A) | mandatory | mandatory | mandatory | best-practice |
| 2.4.3 Focus Order (A) | mandatory | mandatory | mandatory | best-practice |
| 2.4.4 Link Purpose (A) | mandatory | mandatory | mandatory | best-practice |
| 2.4.7 Focus Visible (AA) | mandatory | mandatory | mandatory | best-practice |
| 2.4.11 Focus Not Obscured (AA) | mandatory (EN 301 549:2024) | best-practice | best-practice | best-practice |
| 2.5.3 Label in Name (A) | mandatory | mandatory | mandatory | best-practice |
| 2.5.7 Dragging Movements (AA) | mandatory (EN 301 549:2024) | best-practice | best-practice | best-practice |
| 2.5.8 Target Size Minimum (AA) | mandatory (EN 301 549:2024) | best-practice | best-practice | best-practice |
| 3.1.1 Language of Page (A) | mandatory | mandatory | mandatory | best-practice |
| 3.3.1 Error Identification (A) | mandatory | mandatory | mandatory | best-practice |
| 3.3.2 Labels or Instructions (A) | mandatory | mandatory | mandatory | best-practice |
| 4.1.1 Parsing (A) | mandatory | mandatory | mandatory | best-practice |
| 4.1.2 Name, Role, Value (A) | mandatory | mandatory | mandatory | best-practice |
| 4.1.3 Status Messages (A) | mandatory | mandatory | mandatory | best-practice |

> \* Ontario live captions: mandatory for organizations with 50+ employees; exempt for < 50 employees per IASR O.Reg.191/11 s.14(6)

---

## Multi-Jurisdiction Guidance

When a site must comply with multiple Canadian jurisdictions simultaneously:

1. **Apply the most stringent WCAG version as the floor** — if Federal (WCAG 2.1 AA) and Ontario (WCAG 2.0 AA) both apply, target WCAG 2.1 AA across the board
2. **Federal + Ontario is the most common combination** for large private sector organizations operating in Ontario under federal regulation (e.g., banks with Ontario customers)
3. **Quebec government sites** are typically Quebec-only scope; apply SGQRI 008-02 + WCAG 2.0 AA
4. **Quebec private sector** has no WCAG mandate; cite the Charter and recommend WCAG 2.0 AA as the accommodation floor
5. **Best practice stack** (covers all Canadian jurisdictions): WCAG 2.1 AA (Federal floor) + WCAG 2.2 AA new SCs as enhancements
6. When a SC is mandatory in any applicable jurisdiction, mark it `mandatory`; if only best-practice in all applicable jurisdictions, mark `best-practice`

---

## Selecting Jurisdiction in the Audit

| `--jurisdiction` value | Scope applied |
|----------------------|---------------|
| `global` | WCAG 2.2 only; no jurisdiction citations |
| `federal` | Federal (ACA + CAN/ASC-EN 301 549:2024 → WCAG 2.1 AA) |
| `ontario` | Ontario (AODA + IASR → WCAG 2.0 AA) + best-practice flags for 2.1/2.2 SCs |
| `quebec` | Government scope: SGQRI 008-02 → WCAG 2.0 AA; Private scope: Charter s.10 + WCAG 2.0 AA recommended |

When `--jurisdiction global` is used, all SC-level signals are classified as `best-practice` and no law citations are added.
