# WordPress Development

Practical references for managing WordPress with macOS, MAMP, MySQL, WP-CLI, and DigitalOcean Ubuntu/LAMP.

## Guides

1. [WordPress + MAMP](wordpress-mamp.md): download a site, import its database locally, replace URLs, and cut over to a new production database.
2. [WordPress + WP-CLI](wordpress-wpcli.md): configure local executables, inspect WordPress, check databases, and troubleshoot permalinks.
3. [WordPress audit](wordpress-audit.md): inspect compatibility, security, dependencies, and database structure before proposing changes.

## Workflow

1. Back up the website files and database.
2. Restore the site in MAMP and verify it locally.
3. Audit the installation and review recommendations.
4. Implement approved changes and test locally, subject to the plugin restrictions below.
5. Follow the production cutover procedure when ready to deploy the database.

## Conventions

Replace `<placeholders>` before running commands. Use the same value wherever a placeholder repeats. Database names should use letters, numbers, and underscores. Paths beginning `/<...>` are absolute paths; local projects use `/Applications/MAMP/htdocs/<site>`.

Commands are labeled **local terminal**, **remote terminal**, or **MySQL prompt**. File edits and browser checks are labeled separately. Back up before changes; keep credentials and SQL exports outside Git and database backups outside web roots.

**Codex plugin restriction:** plugin inspection is read-only. Codex must never modify installed plugin source or execute plugin-management commands. Manual reference commands do not grant permission. See the [full restriction](wordpress-audit.md#plugin-restriction--audits-and-future-implementation).
