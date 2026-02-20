---
name: ddev-wordpress
description: "DDEV local environment reference for Périmètre WordPress projects. Covers database import, domain search-replace with cache flush, user create/update, and daily DDEV commands (start, stop, launch, verify)."
tools:
  - Bash
---

# DDEV WordPress Reference

## Basic Commands

```bash
ddev start          # Start the project
ddev stop           # Stop the project
ddev restart        # Restart all containers
ddev launch         # Open the site in browser
ddev launch wp-admin  # Open wp-admin in browser
ddev describe       # Show project URLs, services, and status
```

## Database Import

Always export a backup before importing:

```bash
# 1. Export current DB as backup
ddev export-db --file=backup-$(date +%Y%m%d).sql.gz

# 2. Import new DB
ddev import-db --file=path/to/dump.sql.gz
```

Supported formats: `.sql`, `.sql.gz`, `.zip` (containing a `.sql` file).

## Domain Search-Replace + Cache Flush

Use the `//` prefix to catch both `http://` and `https://` in a single pass.

```bash
# 1. Preview changes (dry run)
ddev wp search-replace '//old-domain.com' '//new-domain.com' --dry-run

# 2. Run the actual replace
ddev wp search-replace '//old-domain.com' '//new-domain.com'

# 3. Flush rewrite rules (also fixes WPGraphQL endpoint issues)
ddev wp rewrite flush
```

> Run `ddev wp cache flush` as well if the site uses an object cache.

## User Management

### Create a user

```bash
ddev wp user create <username> <email> \
  --role=administrator \
  --user_pass=<password>
```

### Update an existing user

```bash
# Update password
ddev wp user update <username-or-id> --user_pass=<new-password>

# Update role
ddev wp user update <username-or-id> --role=editor

# Update email
ddev wp user update <username-or-id> --user_email=new@email.com
```

### List users (helpful to find IDs)

```bash
ddev wp user list
```
