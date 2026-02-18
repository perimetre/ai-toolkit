# Périmètre Design System Review

Provide a comprehensive design system review for recent changes to `@perimetre/ui`.

## Arguments

| Invocation | Mode | Meaning |
|---|---|---|
| _(no args)_ | **changes** | Review current uncommitted changes (default) |
| `--pr` | **pr** | Review the PR for the current branch (auto-detect) |
| `--pr 123` or just `123` | **pr** | Review PR #123 specifically |
| `src/components/` _(any path)_ | **path** | Review all code under that directory |
| `--all` | **path** | Review the entire codebase |

Disambiguation: bare integer → PR number; string with `/` or filesystem path → path mode.

Follow these steps **precisely**:

## Step 0: Determine Review Mode

Check in this order — first match wins:

1. **GitHub context**: Is a PR diff already present in the current conversation or environment (e.g., invoked as a GitHub PR review bot, or the user pasted/attached a diff)? → set mode to `github-pr`, proceed directly to Step 2 (skip Step 1 entirely)
2. **Explicit args**: Classify the argument:
   - `--pr` alone → `pr` mode (auto-detect current branch PR)
   - `--pr 123` or bare integer `123` → `pr` mode for PR #123
   - Any path (contains `/` or is a recognizable filesystem path) → `path` mode
   - `--all` → `path` mode for the entire codebase
3. **Default**: No args and no ambient context → `changes` mode

> **Note:** Always use `gh pr diff` for PR mode — never `git diff main...HEAD`, which incorrectly assumes the base branch.

## Step 1: Gather Review Scope

**Skip this step entirely in `github-pr` mode** — the diff is already in context.

Use a Haiku agent to gather scope based on mode:

- **changes mode**: Run `git status`, `git diff`, and `git log --oneline -5`; return a summary of changed files (components, brand CSS, token CSS, rule files) and what areas they involve
- **pr mode**: Run `gh pr diff [number]` and `gh pr view [number] --json title,body,baseRefName,headRefName`; return a summary of the PR diff and metadata
- **path mode**: Run `find [path] -type f` filtered to relevant extensions (`.ts`, `.tsx`, `.css`, `.json`) — or `git ls-files` for `--all`; return a list of files to review (agents will read them directly in Step 2)

## Step 2: Independent Code Review by Multiple Agents

Launch parallel agents based on mode:

**`github-pr` mode, `changes` mode, or `pr` mode** (diff available):

Launch **3 parallel agents**:
- Use the `component-architect` agent to audit for architecture violations (component structure, CVA patterns, styling rules) and token violations (three-tier system, naming conventions, pragmatic creation, synthetic tokens)
- Use the `brand-reviewer` agent to audit the brand system (CSS architecture, variant composition, new brand checklists)
- Use the `accessibility-reviewer` agent to audit for accessibility violations (ARIA, keyboard navigation, focus management, form fields, contrast, touch targets, reduced motion)

**`path` mode** (no diff — agents read files directly):

Launch **3 parallel agents** (all agents apply — no history agent to skip):
- Use the `component-architect` agent to audit the code at the given path for architecture and token violations; instruct it to read files directly and **cap findings at 5 most critical issues**
- Use the `brand-reviewer` agent to audit the code at the given path for brand system issues; instruct it to read files directly and **cap findings at 5 most critical issues**
- Use the `accessibility-reviewer` agent to audit the code at the given path for accessibility violations; instruct it to read files directly and **cap findings at 5 most critical issues**

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

**Mode:** [Changes | PR #123 | Path: src/components/ | All code]

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

**For `changes` and `pr` mode:**

- Pre-existing issues not introduced in this diff
- One-off hardcoded values in consumer app code (not inside `packages/ui`)
- External consumer code that uses the library but doesn't follow internal authoring patterns
- Deprecated patterns that have already been replaced in the same diff
- `outline-none` when a custom `focus-visible` ring is present on the same element
- Brand variant files that intentionally duplicate Acorn styles during a migration in progress (check git log for intent)
- Token names that differ from convention due to a pending rename tracked in a separate issue

**For `path` mode:**

- Pre-existing issues are fair game — reviewing existing code at a path is the point
- Still exclude: one-off hardcoded values in consumer app code (not inside `packages/ui`)
- Still exclude: issues linters/typecheckers would catch automatically
- Still exclude: issues explicitly silenced in code (lint ignore comments)
- Still exclude: pedantic nitpicks a senior engineer wouldn't call out

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
