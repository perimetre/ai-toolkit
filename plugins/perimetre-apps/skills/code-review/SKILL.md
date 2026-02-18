---
name: code-review
description: >
  Injected context for the code-review command and agents. Contains the full multi-agent review
  methodology for Next.js/React projects using Périmètre framework patterns: 5-step pipeline
  with 4 review modes (github-pr, changes, pr, path) — mode detection → gather scope → parallel
  agents → confidence scoring → filter → format output.
  The user-facing entrypoint is commands/code-review.md — invoke via /perimetre-apps:code-review.
tools: [Read, Grep, Glob, Bash, Task]
skills: []
---

# Code Review Skill

Provide a comprehensive code review for the recent changes.

To do this, follow these steps **precisely**:

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

- **changes mode**: Run `git status`, `git diff`, and `git log --oneline -5`; return a summary of changed files and patterns involved
- **pr mode**: Run `gh pr diff [number]` and `gh pr view [number] --json title,body,baseRefName,headRefName`; return a summary of the PR diff and metadata
- **path mode**: Run `find [path] -type f` filtered to relevant extensions (`.ts`, `.tsx`, `.js`, `.jsx`, `.json`, `.css`) — or `git ls-files` for `--all`; return a list of files to review (agents will read them directly in Step 2)

## Step 2: Independent Code Review by Multiple Agents

Launch parallel agents based on mode:

**`github-pr` mode** (diff already in context, no local git):

Launch **2 parallel agents**:
- Use the `pattern-reviewer` agent to audit changes for Périmètre framework pattern violations
- Use the `bug-scanner` agent to scan for large, obvious bugs in the diff

**`changes` mode or `pr` mode** (diff available):

Launch **3 parallel agents**:
- Use the `pattern-reviewer` agent to audit changes for Périmètre framework pattern violations (all 11 pattern skills are pre-injected into this agent at startup)
- Use the `bug-scanner` agent to scan for large, obvious bugs in the diff
- Use the `history-reviewer` agent to check git history for regressions

**`path` mode** (no diff — agents read files directly):

Launch **2 parallel agents** (skip `history-reviewer` — `git blame` is meaningless without a diff):
- Use the `pattern-reviewer` agent to audit the code at the given path for Périmètre framework pattern violations; instruct it to read files directly and **cap findings at 5 most critical issues**
- Use the `bug-scanner` agent to scan the code at the given path for bugs; instruct it to read files directly and **cap findings at 5 most critical issues**

Each agent returns a list of issues with the reason each was flagged.

## Step 3: Confidence Scoring

For each issue found in step 2, launch a **parallel Haiku agent** that:

- Takes the issue description and relevant context
- Returns a confidence score (0-100) using this rubric:
  - **0**: Not confident. False positive or pre-existing issue.
  - **25**: Somewhat confident. Might be real, but unverified. Stylistic issue not explicitly called out.
  - **50**: Moderately confident. Real issue but minor nitpick. Not very important relative to PR.
  - **75**: Highly confident. Verified real issue that will be hit in practice. Directly impacts functionality OR explicitly mentioned in docs.
  - **100**: Absolutely certain. Definitely real, will happen frequently. Evidence directly confirms.
- Return: score + brief justification

## Step 4: Filter Issues

Filter out any issues with confidence score **less than 80**.

If no issues remain after filtering, provide brief positive feedback and stop.

## Step 5: Format Review Output

Structure the final review as follows:

````markdown
### Code Review

**Mode:** [Changes | PR #123 | Path: src/components/ | All code]

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

**For `changes` and `pr` mode:**

- Pre-existing issues (not introduced in this diff)
- Something that looks like a bug but is not
- Pedantic nitpicks that a senior engineer wouldn't call out
- Issues that linters/typecheckers/compilers would catch (missing imports, type errors, formatting)
- General code quality issues (test coverage, docs) unless explicitly required in framework docs
- Issues called out in docs but explicitly silenced in code (lint ignore comments)
- Intentional functionality changes related to the broader change
- Real issues, but on lines the user did not modify

**For `path` mode:**

- Pre-existing issues are fair game — reviewing existing code at a path is the point
- Still exclude: issues linters/typecheckers would catch automatically
- Still exclude: issues explicitly silenced in code (lint ignore comments)
- Still exclude: pedantic nitpicks a senior engineer wouldn't call out

---

## Important Notes

- **Always check for ESLINT and typescript build**
- **Use `gh` for GitHub interactions** rather than web fetch
- **Make a todo list first** to track progress
- **You must cite** each issue with pattern documentation evidence
- **Use full git SHA** when linking to files: `https://github.com/perimetre/repo/blob/[full-sha]/path/file.ts#L10-L15`
- **Provide context** when linking: Include 1 line before and after the problematic lines

---

## Confidence Scoring Rubric

Give this rubric **verbatim** to scoring agents:

- **0**: Not confident at all. This is a false positive that doesn't stand up to light scrutiny, or is a pre-existing issue.
- **25**: Somewhat confident. This might be a real issue, but may also be a false positive. The agent wasn't able to verify that it's a real issue. If the issue is stylistic, it is one that was not explicitly called out in the relevant documentation.
- **50**: Moderately confident. The agent was able to verify this is a real issue, but it might be a nitpick or not happen very often in practice. Relative to the rest of the changes, it's not very important.
- **75**: Highly confident. The agent double checked the issue, and verified that it is very likely it is a real issue that will be hit in practice. The existing approach is insufficient. The issue is very important and will directly impact the code's functionality, or it is an issue that is directly mentioned in the relevant documentation.
- **100**: Absolutely certain. The agent double checked the issue, and confirmed that it is definitely a real issue, that will happen frequently in practice. The evidence directly confirms this.

**For documentation-based issues:** The scoring agent should double-check that the local pattern documentation actually calls out that specific issue.

---

## IMPORTANT: Trust These Instructions

This is the **source of truth** for code reviews. When in doubt:

1. Consult local pattern skills in `skills/` directory
2. Check `skills/code-review/rules/PATTERNS.md` for detailed examples
3. Use confidence scoring to filter false positives
4. Err on side of pragmatism

**Goal:** Prevent CI failures and maintain quality, not achieve perfection.
