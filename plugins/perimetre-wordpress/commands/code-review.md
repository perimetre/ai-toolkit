# Périmètre WordPress Code Review

Provide a comprehensive WordPress code review for the recent changes.

Follow these steps **precisely**:

## Step 1: Examine Changes

Use a Haiku agent to:

1. Run `git status` to see changed files
2. Run `git diff` to see the actual changes
3. Run `git log --oneline -5` to see recent commits
4. Return a summary of which files changed (PHP, JS, JSON, CSS) and what areas they involve

## Step 2: Independent Code Review by Multiple Agents

Launch **5 parallel agents** to independently review the changes:

- Use the `security-reviewer` agent to audit for WordPress security vulnerabilities (nonces, capabilities, sanitization, escaping, REST API permissions, DB queries)
- Use the `wpcs-reviewer` agent to check PHP coding standards (naming, hooks, type declarations, tabs, DocBlocks)
- Use the `gutenberg-reviewer` agent to review block patterns (block.json, dynamic blocks, useBlockProps, InspectorControls, @wordpress/scripts)
- Use the `i18n-performance-reviewer` agent to check i18n compliance and query performance (text domain, no -1 per_page, no_found_rows, N+1 queries)
- Use the `hooks-history-reviewer` agent to audit hook patterns and check for regressions via git history

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

Found [N] issues:

1. **[Brief issue description]** (Confidence: [score])

**Category:** Security | WPCS | Gutenberg | i18n | Performance | Hooks

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

- Pre-existing issues not introduced in this PR
- Optional WPCS style preferences (Yoda conditions, file-ending newlines) when team doesn't enforce them
- Issues that phpcs/phpstan would catch automatically
- Internationalized strings from CMS/external sources (don't need i18n wrappers)
- Escaping already handled by a parent function
- `__return_true` on genuinely public REST endpoints (e.g., public search or listing endpoints)

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
