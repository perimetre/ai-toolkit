---
name: code-review
description: >
  Injected context for the WordPress code-review command and agents. Contains the full
  multi-agent review methodology for PHP themes, Gutenberg blocks, custom plugins, and
  REST API extensions: security (nonces, sanitization, escaping, capabilities), WPCS,
  i18n, query performance, and hook patterns. The user-facing entrypoint is
  commands/code-review.md — invoke via /perimetre-wordpress:code-review.
allowed-tools: Read, Grep, Glob, Bash, Task
---

# WordPress Code Review

Provide a comprehensive WordPress code review for the recent changes.

Follow these steps **precisely**:

## Step 1: Examine Changes

Use a Haiku agent to:

1. Run `git status` to see changed files
2. Run `git diff` to see the actual changes
3. Run `git log --oneline -5` to see recent commits
4. Return a summary of:
   - Which files changed (PHP, JS, JSON, CSS)
   - What areas they involve (theme setup, blocks, REST API, CPTs, hooks, queries)
   - Which review areas are most relevant

## Step 2: Independent Code Review by Multiple Agents

Launch **5 parallel Sonnet agents** to independently review the changes:

**Agent #1: Security Audit**

Review for WordPress security vulnerabilities:

- **Nonces** — Forms use `wp_nonce_field()`, AJAX uses `check_ajax_referer()`, admin actions use `check_admin_referer()`
- **Capability checks** — All write/admin operations call `current_user_can()` before proceeding
- **Input sanitization** — Every `$_POST`, `$_GET`, `$_REQUEST` value is sanitized with the appropriate function:
  - `sanitize_text_field()` for general text
  - `absint()` for integers
  - `sanitize_email()` for email addresses
  - `esc_url_raw()` for URLs stored to DB
  - `wp_kses_post()` for HTML content
- **Output escaping** — Every value echoed to the page is escaped:
  - `esc_html()` for plain text
  - `esc_attr()` for HTML attributes
  - `esc_url()` for URLs in output
  - `wp_kses_post()` for post content
- **REST API permissions** — All non-public endpoints have a proper `permission_callback` (never `__return_true` on write endpoints)
- **Direct DB queries** — All `$wpdb->prepare()` with placeholders, never string concatenation with user input

**Agent #2: PHP Coding Standards (WPCS)**

Review for WordPress PHP coding standards:

- **Naming** — Functions use `snake_case` with theme/plugin prefix; classes use `PascalCase`
- **Hooks** — All hooks registered inside functions, not at file scope; priority and `$accepted_args` always explicit
- **Yoda conditions** — `if ( 'value' === $variable )` not `if ( $variable === 'value' )` (optional but consistent)
- **Type declarations** — PHP 7.4+ type hints on function arguments and return types
- **No closing PHP tag** — PHP files must not have `?>` at end
- **Tabs not spaces** — WordPress uses tabs for indentation in PHP
- **Documentation** — Public functions have DocBlocks (`@param`, `@return`)
- **`wpdb` globals** — Access via `global $wpdb;` not direct `$GLOBALS`
- **Deprecated functions** — No use of deprecated WP functions (check against current WP version)

**Agent #3: Gutenberg & Block Patterns**

Review block-related code:

- **`block.json`** — Present for all blocks; includes `apiVersion: 3`, `name` with namespace, `editorScript`/`style` references
- **Dynamic blocks** — `save()` returns `null` when rendering in PHP; PHP render callback uses `esc_html()` on all output
- **Block registration** — Uses `register_block_type( __DIR__ )` pointing to block.json directory
- **Block attributes** — No logic in `block.json` defaults that belongs in PHP
- **`useBlockProps()`** — Spread in both `edit` and `save` (or `useBlockProps.save()` in static blocks)
- **`InspectorControls`** — All block settings in sidebar panel, not inline
- **`@wordpress/scripts`** — Build process uses `@wordpress/scripts`, not custom webpack
- **No inline styles** — Block styles via `style.css` / `editor.css`, not inline in render

**Agent #4: i18n & Query Performance**

Review internationalization and database usage:

*i18n checks:*
- All user-facing strings wrapped in `__()`, `_e()`, `esc_html__()`, or `_n()`
- Consistent text domain used throughout (matches `Text Domain` in style.css / plugin header)
- No string concatenation across translated strings — use `printf()` / `sprintf()`
- `load_theme_textdomain()` or `load_plugin_textdomain()` called in `after_setup_theme` / `init`

*Query performance checks:*
- No `posts_per_page => -1` on CPT queries — always paginate or use reasonable limits
- `no_found_rows => true` when pagination is not needed (skips expensive COUNT query)
- `update_post_meta_cache => false` / `update_post_term_cache => false` when meta/terms not needed
- No direct DB queries (`$wpdb->get_results`) where `WP_Query` or core APIs suffice
- No N+1 queries in loops (e.g., calling `get_post_meta()` inside `foreach` over posts)

**Agent #5: Hook Patterns & Historical Context**

Review hook usage and code history:

*Hook patterns:*
- `add_action` / `add_filter` called with all four arguments (hook, callback, priority, accepted_args) for non-default values
- Hooks registered in the correct lifecycle (e.g., `init` for CPTs, `rest_api_init` for REST routes, `wp_enqueue_scripts` for assets)
- Callbacks are prefixed functions, not closures (closures cannot be removed with `remove_action`)
- `remove_action` / `remove_filter` uses matching priority
- No `add_filter` inside another filter callback (causes infinite loops)

*Historical context:*
- Read git blame and history of modified files
- Check for regressions — does the change break existing behavior?
- Look for patterns of repeated issues in previous commits

## Step 3: Confidence Scoring

For each issue found, launch a **parallel Haiku agent** that assigns a confidence score (0-100):

- **0**: False positive or pre-existing issue not introduced in this diff
- **25**: Might be real but unverified; stylistic concern not in WPCS
- **50**: Real issue but minor nitpick; limited practical impact
- **75**: Verified real issue that will be hit in practice; security risk or functional problem
- **100**: Definite vulnerability or broken functionality; confirmed by diff evidence

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
