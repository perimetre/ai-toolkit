# Périmètre Design System Review

Provide a comprehensive design system review for recent changes to `@perimetre/ui`.

Follow these steps **precisely**:

## Step 1: Examine Changes

Use a Haiku agent to:

1. Run `git status` to see changed files
2. Run `git diff` to see the actual changes
3. Run `git log --oneline -5` to see recent commits
4. Return a summary of which files changed (components, brand CSS, token CSS, rule files) and what areas they involve

## Step 2: Independent Code Review by Multiple Agents

Launch **3 parallel agents** to independently review the changes:

- Use the `component-architect` agent to audit for architecture violations (component structure, CVA patterns, styling rules) and token violations (three-tier system, naming conventions, pragmatic creation, synthetic tokens)
- Use the `brand-reviewer` agent to audit the brand system (CSS architecture, variant composition, new brand checklists)
- Use the `accessibility-reviewer` agent to audit for accessibility violations (ARIA, keyboard navigation, focus management, form fields, contrast, touch targets, reduced motion)

Each agent returns a list of issues with the reason each was flagged.

## Step 3: Confidence Scoring

For each issue found in Step 2, launch a **parallel Haiku agent** that assigns a confidence score (0-100):

- **0**: Not confident at all. This is a false positive that doesn't stand up to light scrutiny, or is a pre-existing issue.
- **25**: Somewhat confident. This might be a real issue, but may also be a false positive. The agent wasn't able to verify that it's a real issue. If the issue is stylistic, it is one that was not explicitly called out in the relevant documentation.
- **50**: Moderately confident. The agent was able to verify this is a real issue, but it might be a nitpick or not happen very often in practice. Relative to the rest of the changes, it's not very important.
- **75**: Highly confident. The agent double checked the issue, and verified that it is very likely it is a real issue that will be hit in practice. The existing approach is insufficient. The issue is very important and will directly impact the code's functionality, or it is an issue that is directly mentioned in the relevant documentation.
- **100**: Absolutely certain. The agent double checked the issue, and confirmed that it is definitely a real issue, that will happen frequently in practice. The evidence directly confirms this.

## Step 4: Filter Issues

Filter out any issues with confidence score **less than 80**.

If no issues remain after filtering, provide brief positive feedback and stop.

## Step 5: Format Review Output

````markdown
### Design System Review

Found [N] issues:

1. **[Brief issue description]** (Confidence: [score])

**Category:** Architecture | Tokens | Brand | Accessibility | Integration

**File:** `[path/to/file.tsx:line]`

**Current:**

```tsx
[problematic code]
```

**Fix:**

```tsx
[corrected code]
```

**Why:** [Impact statement — broken brand switching, missing token, inaccessible pattern, etc.]

**Reference:** `skills/design-system/rules/[rule-file].md`

---

2. [Next issue...]

---

### Summary

**Recommendation:** [APPROVE / REQUEST CHANGES]

**Issues by category:**

- Architecture: [count]
- Tokens: [count]
- Brand: [count]
- Accessibility: [count]
- Integration: [count]

**Next steps:**

1. [Action item]
````

---

## Examples of False Positives (DO NOT FLAG)

- Pre-existing issues not introduced in this diff
- One-off hardcoded values in consumer app code (not inside `packages/ui`)
- External consumer code that uses the library but doesn't follow internal authoring patterns
- Deprecated patterns that have already been replaced in the same diff
- `outline-none` when a custom `focus-visible` ring is present on the same element
- Brand variant files that intentionally duplicate Acorn styles during a migration in progress (check git log for intent)
- Token names that differ from convention due to a pending rename tracked in a separate issue

---

## Confidence Scoring Rubric

Give this rubric **verbatim** to scoring agents:

- **0**: Not confident at all. This is a false positive that doesn't stand up to light scrutiny, or is a pre-existing issue.
- **25**: Somewhat confident. This might be a real issue, but may also be a false positive. The agent wasn't able to verify that it's a real issue. If the issue is stylistic, it is one that was not explicitly called out in the relevant documentation.
- **50**: Moderately confident. The agent was able to verify this is a real issue, but it might be a nitpick or not happen very often in practice. Relative to the rest of the changes, it's not very important.
- **75**: Highly confident. The agent double checked the issue, and verified that it is very likely it is a real issue that will be hit in practice. The existing approach is insufficient. The issue is very important and will directly impact the code's functionality, or it is an issue that is directly mentioned in the relevant documentation.
- **100**: Absolutely certain. The agent double checked the issue, and confirmed that it is definitely a real issue, that will happen frequently in practice. The evidence directly confirms this.

---

## Reference

Local rule files (for deep-dive on specific patterns):

- `skills/design-system/rules/architecture-component-structure.md`
- `skills/design-system/rules/architecture-cva-patterns.md`
- `skills/design-system/rules/architecture-styling-rules.md`
- `skills/design-system/rules/tokens-three-tier-system.md`
- `skills/design-system/rules/tokens-naming-conventions.md`
- `skills/design-system/rules/tokens-pragmatic-creation.md`
- `skills/design-system/rules/tokens-synthetic-tokens.md`
- `skills/design-system/rules/brands-adding-new.md`
- `skills/design-system/rules/brands-css-architecture.md`
- `skills/design-system/rules/brands-variant-composition.md`
- `skills/design-system/rules/accessibility-patterns.md`
- `skills/design-system/rules/consumption-integration.md`
