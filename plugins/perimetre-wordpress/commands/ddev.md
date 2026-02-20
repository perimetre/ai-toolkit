---
description: "Run common DDEV operations: db import, domain replace, user management, or basic commands."
argument-hint: "[import | replace | user | help]"
allowed-tools:
  - Bash
---

## DDEV Reference

!`cat "${CLAUDE_PLUGIN_ROOT}/skills/ddev-wordpress/SKILL.md"`

---

Detect the mode from `$ARGUMENTS`:

- `import` → DB import workflow
- `replace` → Domain search-replace + flush workflow
- `user` → User create/update workflow
- `help` or no args → Show available modes

---

## Mode: no args or `help`

Show this summary and stop:

```
/perimetre-wordpress:ddev — DDEV operations

  import   Import a database file
  replace  Search-replace a domain across the DB + flush rewrites
  user     Create or update a WordPress user
  help     Show this message
```

---

## Mode: `import`

1. Ask the user: "Path to the database file to import?"

2. Run the import:
   ```
   ddev import-db --file=<path>
   ```

3. Report success or surface the error output.

---

## Mode: `replace`

1. Ask the user: "Old domain to replace? (e.g. old-site.local)"

2. Ask the user: "New domain? (e.g. new-site.ddev.site)"

3. Run a dry-run and show the output:
   ```
   ddev wp search-replace '//old-domain' '//new-domain' --all-tables-with-prefix --dry-run
   ```

4. Ask: "Looks good? Run the actual replace? (yes/no)"

5. If yes:
   ```
   ddev wp search-replace '//old-domain' '//new-domain' --all-tables-with-prefix
   ddev wp rewrite flush
   ```

6. Confirm completion and remind the user to test the site with `ddev launch`.

---

## Mode: `user`

1. Ask: "Create a new user or update an existing one? (create/update)"

**If create:**
- Ask for: username, email, role (default: administrator), password
- Run:
  ```
  ddev wp user create <username> <email> --role=<role> --user_pass=<password>
  ```

**If update:**
- Ask: "Username or user ID?"
- Ask: "What to update? (password / role / email)"
- Run the matching `ddev wp user update` command

Show the result and confirm success.
