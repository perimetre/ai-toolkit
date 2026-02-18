# Périmètre Code Review

Provide a comprehensive code review for the recent changes using Périmètre framework patterns.

Follow these steps **precisely**:

## Step 1: Examine Changes

Use a Haiku agent to:

1. Run `git status` to see changed files
2. Run `git diff` to see the actual changes
3. Run `git log --oneline -5` to see recent commits
4. Return a summary of which files changed and what patterns they involve

## Step 2: Independent Code Review by Multiple Agents

Launch **3 parallel agents** to independently review the changes:

- Use the `pattern-reviewer` agent to audit changes for Périmètre framework pattern violations
- Use the `bug-scanner` agent to scan for large, obvious bugs in the diff
- Use the `history-reviewer` agent to check git history for regressions

Each agent returns a list of issues with the reason each was flagged.

## Step 3: Confidence Scoring

For each issue found in Step 2, launch a **parallel Haiku agent** that:

- Takes the issue description and relevant context
- Returns a confidence score (0-100) using this rubric:
  - **0**: Not confident at all. This is a false positive that doesn't stand up to light scrutiny, or is a pre-existing issue.
  - **25**: Somewhat confident. This might be a real issue, but may also be a false positive. The agent wasn't able to verify that it's a real issue. If the issue is stylistic, it is one that was not explicitly called out in the relevant documentation.
  - **50**: Moderately confident. The agent was able to verify this is a real issue, but it might be a nitpick or not happen very often in practice. Relative to the rest of the changes, it's not very important.
  - **75**: Highly confident. The agent double checked the issue, and verified that it is very likely it is a real issue that will be hit in practice. The existing approach is insufficient. The issue is very important and will directly impact the code's functionality, or it is an issue that is directly mentioned in the relevant documentation.
  - **100**: Absolutely certain. The agent double checked the issue, and confirmed that it is definitely a real issue, that will happen frequently in practice. The evidence directly confirms this.
- Return: score + brief justification

## Step 4: Filter Issues

Filter out any issues with confidence score **less than 80**.

If no issues remain after filtering, provide brief positive feedback and stop.

## Step 5: Format Review Output

Structure the final review as follows:

````markdown
### Code Review

Found [N] issues:

1. **[Brief issue description]** (Confidence: [score])

**Evidence:** According to `[skill-name]`:

> [Quote from pattern documentation]

**File:** `[path/to/file.ts:lines]`

**Current:**

```[lang]
[problematic code]
```

**Fix:**

```[lang]
[corrected code]
```

**Why:** [Impact statement]

**Reference:** `skills/[skill-name]/SKILL.md`

---

2. [Next issue...]

---

### Summary

**Recommendation:** [APPROVE / REQUEST CHANGES]

**Issues by severity:**

- Critical: [count]
- High: [count]
- Medium: [count]

**Next steps:**

1. [Action item]
2. [Action item]
````

**Output Requirements:**

- Keep output concise and actionable
- Cite specific pattern documentation
- Provide code examples for every issue
- Include confidence scores
- Avoid emojis (except 🤖 in footer)

**If no issues found:**

```markdown
### Code Review

No issues found. Checked for:

- Framework pattern compliance ([list patterns checked])
- Obvious bugs
- Historical context issues

**Recommendation:** APPROVE
```

---

## Examples of False Positives (DO NOT FLAG)

- Pre-existing issues (not introduced in this PR)
- Something that looks like a bug but is not
- Pedantic nitpicks that a senior engineer wouldn't call out
- Issues that linters/typecheckers/compilers would catch (missing imports, type errors, formatting)
- General code quality issues (test coverage, docs) unless explicitly required in framework docs
- Issues called out in docs but explicitly silenced in code (lint ignore comments)
- Intentional functionality changes related to the broader change
- Real issues, but on lines the user did not modify

---

## Confidence Scoring Rubric

Give this rubric **verbatim** to scoring agents:

- **0**: Not confident at all. This is a false positive that doesn't stand up to light scrutiny, or is a pre-existing issue.
- **25**: Somewhat confident. This might be a real issue, but may also be a false positive. The agent wasn't able to verify that it's a real issue. If the issue is stylistic, it is one that was not explicitly called out in the relevant documentation.
- **50**: Moderately confident. The agent was able to verify this is a real issue, but it might be a nitpick or not happen very often in practice. Relative to the rest of the changes, it's not very important.
- **75**: Highly confident. The agent double checked the issue, and verified that it is very likely it is a real issue that will be hit in practice. The existing approach is insufficient. The issue is very important and will directly impact the code's functionality, or it is an issue that is directly mentioned in the relevant documentation.
- **100**: Absolutely certain. The agent double checked the issue, and confirmed that it is definitely a real issue, that will happen frequently in practice. The evidence directly confirms this.

---

## Important Notes

- **Always check for ESLINT and typescript build**
- **Use `gh` for GitHub interactions** rather than web fetch
- **Make a todo list first** to track progress
- **You must cite** each issue with pattern documentation evidence
- **Use full git SHA** when linking to files: `https://github.com/perimetre/repo/blob/[full-sha]/path/file.ts#L10-L15`
- **Provide context** when linking: Include 1 line before and after the problematic lines

---

## Reference

Local skill files (for deep-dive on specific patterns):

- `skills/error-handling/SKILL.md`
- `skills/services/SKILL.md`
- `skills/trpc/SKILL.md`
- `skills/forms/SKILL.md`
- `skills/graphql/SKILL.md`
- `skills/tanstack-query/SKILL.md`
- `skills/icons/SKILL.md`
- `skills/images/SKILL.md`
- `skills/seo/SKILL.md`
- `skills/caching/SKILL.md`
- `skills/payload-drizzle/SKILL.md`
- `skills/code-review/rules/PATTERNS.md` (detailed examples)
