---
description: "WordPress code review: security, WPCS, Gutenberg, i18n, query performance, and HTML standards. Supports changes, PR, path, and GitHub PR modes."
argument-hint: "[--pr [number] | <path> | --all]"
allowed-tools: [Read, Grep, Glob, Bash, Task]
---

# Périmètre WordPress Code Review

Provide a comprehensive WordPress code review for the recent changes.

## Arguments

| Invocation | Mode | Meaning |
|---|---|---|
| _(no args)_ | **changes** | Review current uncommitted changes (default) |
| `--pr` | **pr** | Review the PR for the current branch (auto-detect) |
| `--pr 123` or just `123` | **pr** | Review PR #123 specifically |
| `src/` _(any path)_ | **path** | Review all code under that directory |
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

- **changes mode**: Run `git status`, `git diff`, and `git log --oneline -5`; return a summary of changed files (PHP, JS, JSON, CSS) and what areas they involve
- **pr mode**: Run `gh pr diff [number]` and `gh pr view [number] --json title,body,baseRefName,headRefName`; return a summary of the PR diff and metadata
- **path mode**: Run `find [path] -type f` filtered to relevant extensions (`.php`, `.js`, `.json`, `.css`) — or `git ls-files` for `--all`; return a list of files to review (agents will read them directly in Step 2)

## Step 2: Independent Code Review by Multiple Agents

Launch parallel agents based on mode:

**`github-pr` mode** (diff already in context, no local git):

Launch **5 parallel agents** (skip `hooks-history-reviewer` — no local git):
- Use the `security-reviewer` agent to audit for WordPress security vulnerabilities (nonces, capabilities, sanitization, escaping, REST API permissions, DB queries)
- Use the `wpcs-reviewer` agent to check PHP coding standards (naming, hooks, type declarations, tabs, DocBlocks)
- Use the `gutenberg-reviewer` agent to review block patterns (block.json, dynamic blocks, useBlockProps, InspectorControls, @wordpress/scripts)
- Use the `i18n-performance-reviewer` agent to check i18n compliance and query performance (text domain, no -1 per_page, no_found_rows, N+1 queries)
- Use the `html-standards-reviewer` agent to check HTML coding standards (self-closing elements, lowercase tags/attributes, quoted attributes, boolean attribute values, tab indentation, PHP/HTML mixing alignment)

**`changes` mode or `pr` mode** (diff available):

Launch **6 parallel agents**:
- Use the `security-reviewer` agent to audit for WordPress security vulnerabilities (nonces, capabilities, sanitization, escaping, REST API permissions, DB queries)
- Use the `wpcs-reviewer` agent to check PHP coding standards (naming, hooks, type declarations, tabs, DocBlocks)
- Use the `gutenberg-reviewer` agent to review block patterns (block.json, dynamic blocks, useBlockProps, InspectorControls, @wordpress/scripts)
- Use the `i18n-performance-reviewer` agent to check i18n compliance and query performance (text domain, no -1 per_page, no_found_rows, N+1 queries)
- Use the `hooks-history-reviewer` agent to audit hook patterns and check for regressions via git history
- Use the `html-standards-reviewer` agent to check HTML coding standards (self-closing elements, lowercase tags/attributes, quoted attributes, boolean attribute values, tab indentation, PHP/HTML mixing alignment)

**`path` mode** (no diff — agents read files directly):

Launch **5 parallel agents** (skip `hooks-history-reviewer` — `git blame` is meaningless without a diff):
- Use the `security-reviewer` agent to audit the code at the given path for WordPress security vulnerabilities; instruct it to read files directly and **cap findings at 5 most critical issues**
- Use the `wpcs-reviewer` agent to audit the code at the given path for PHP coding standards; instruct it to read files directly and **cap findings at 5 most critical issues**
- Use the `gutenberg-reviewer` agent to audit the code at the given path for block pattern issues; instruct it to read files directly and **cap findings at 5 most critical issues**
- Use the `i18n-performance-reviewer` agent to audit the code at the given path for i18n and performance issues; instruct it to read files directly and **cap findings at 5 most critical issues**
- Use the `html-standards-reviewer` agent to audit the code at the given path for HTML coding standards violations; instruct it to read files directly and **cap findings at 5 most critical issues**

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

If no issues remain, provide brief positive feedback and stop.

## Step 5: Format Review Output

````markdown
### WordPress Code Review

**Mode:** [Changes | PR #123 | Path: src/ | All code]

Found [N] issues:

1. **[Brief issue description]** (Confidence: [score])

**Category:** Security | WPCS | Gutenberg | i18n | Performance | Hooks | HTML

**File:** `[path/to/file.php:line]`

**Current:**

```php
[problematic code]
```

**Fix:**

```php
[corrected code]
```

**Why:** [Impact statement — security risk, standards violation, performance cost]

---

2. [Next issue...]

---

### Summary

**Recommendation:** [APPROVE / REQUEST CHANGES]

**Issues by category:**

- Security: [count]
- Standards: [count]
- Performance: [count]
- i18n: [count]
- Gutenberg: [count]

**Next steps:**

1. [Action item]
````

---

## Examples of False Positives (DO NOT FLAG)

**For `changes` and `pr` mode:**

- Pre-existing issues not introduced in this PR
- Optional WPCS style preferences (Yoda conditions, file-ending newlines) when team doesn't enforce them
- Issues that phpcs/phpstan would catch automatically
- Internationalized strings from CMS/external sources (don't need i18n wrappers)
- Escaping already handled by a parent function
- `__return_true` on genuinely public REST endpoints (e.g., public search or listing endpoints)

**For `path` mode:**

- Pre-existing issues are fair game — reviewing existing code at a path is the point
- Still exclude: issues phpcs/phpstan would catch automatically
- Still exclude: issues explicitly silenced in code (phpcs ignore comments)
- Still exclude: pedantic nitpicks a senior engineer wouldn't call out
- Still exclude: `__return_true` on genuinely public REST endpoints

---

## Security Quick Reference

| Risk | Insecure | Secure |
|------|----------|--------|
| User input from `$_POST` | `$_POST['field']` | `sanitize_text_field( $_POST['field'] )` |
| Integer input | `$_GET['id']` | `absint( $_GET['id'] )` |
| HTML output | `echo $title` | `echo esc_html( $title )` |
| URL output | `echo $url` | `echo esc_url( $url )` |
| Attribute output | `echo $val` | `echo esc_attr( $val )` |
| DB query | `"WHERE id = $id"` | `$wpdb->prepare( "WHERE id = %d", $id )` |
| Form | (no nonce) | `wp_nonce_field( 'peri_action', 'peri_nonce' )` |
| Admin action | (no cap check) | `if ( ! current_user_can( 'manage_options' ) ) wp_die()` |
