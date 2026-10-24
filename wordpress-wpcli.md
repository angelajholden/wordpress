# WP-CLI with MAMP

All commands below are **local terminal** commands unless labeled otherwise. Replace placeholders using the [repository conventions](README.md#conventions). Back up before commands that change files or data. Database commands use the connection in `wp-config.php`.

## 1. Install WP-CLI

```bash
brew install wp-cli
wp --info
```

## 2. Select MAMP's PHP and MySQL

Find the installed versions, then add their executable directories to the current session's PATH. `<php-directory>` might be `php8.4.1`; `<mysql-directory>` might be `mysql80`. Match MAMP's selected versions.

```bash
ls /Applications/MAMP/bin/php/
find /Applications/MAMP/Library -type f -name mysql
export PATH="/Applications/MAMP/bin/php/<php-directory>/bin:$PATH"
export PATH="/Applications/MAMP/Library/bin/<mysql-directory>/bin:$PATH"
which php
php -v
which mysql
mysql --version
wp --info
```

Confirm `wp --info` reports the intended MAMP executables. Homebrew PHP may differ from MAMP PHP and produce compatibility warnings. These PATH changes last only for this session; add them to `~/.zshrc` only if intentionally making them persistent. The MySQL client version does not identify the running server version.

## 3. Enter the WordPress directory

Run the remaining commands from the installation you intend to manage.

```bash
cd /Applications/MAMP/htdocs/<site>
wp core is-installed
wp core version
```

## 4. Inspect WordPress URLs

```bash
wp option get home
wp option get siteurl
```

For database-wide replacements, follow the [local migration](wordpress-mamp.md#7-replace-urls--local-terminal) or [production cutover](wordpress-mamp.md#production-database-cutover) steps rather than changing URLs independently.

## 5. Plugins — manual reference only

**These commands are for the developer to run manually. They do not authorize Codex to execute any plugin-management command, including the inspection commands below. Codex may inspect plugin files read-only but must never modify installed plugin source. Plugin actions require separate explicit approval.** See the [full restriction](wordpress-audit.md#plugin-restriction--audits-and-future-implementation).

```bash
# Inspect installed plugins and available updates
wp plugin list
wp plugin list --update=available
wp plugin list --status=active

# Update only after reviewing compatibility and making a backup
wp plugin update <plugin-slug>
wp plugin update --all

# Change activation state only when intended
wp plugin activate <plugin-slug>
wp plugin deactivate <plugin-slug>
wp plugin deactivate --all
```

## 6. Themes

Inspect first; activate or update only when intended and backed up.

```bash
wp theme list
wp theme list --update=available
wp theme activate <theme-slug>
wp theme update <theme-slug>
```

## 7. Users

```bash
wp user list
wp user get <wp-username>
wp user list --role=administrator
wp user update <wp-username> --prompt=user_pass
```

## 8. Check the database

Run the integrity and revision checks, then export before making changes. Use a new backup filename to preserve earlier backups.

```bash
wp db check
wp core update-db --dry-run
wp db export /Users/<mac-user>/Downloads/<site>-backup.sql
wp db cli
```

`wp db check` checks MySQL table integrity. `wp core update-db --dry-run` compares WordPress's recorded database revision with installed core; neither proves that every expected column or index exists.

**Local MySQL prompt:** inspect actual structure as needed. Replace `<table-name>` and `<column-name>` with names from this installation, including its actual table prefix.

```sql
SHOW TABLES;
SHOW COLUMNS FROM <table-name>;
SHOW INDEX FROM <table-name>;
SELECT COUNT(*) AS null_count FROM <table-name> WHERE <column-name> IS NULL;
EXIT;
```

Compare definitions and index column order with the installed WordPress version before proposing corrections. An SQL export is a snapshot: reading it does not modify MySQL, and changes to MySQL do not update the export. Export again after changes before deployment.

## 9. Check cache and permalinks

Flush only a local object cache. Read the current permalink structure; change it only if `/%postname%/` is the intended format, then flush rules.

```bash
wp cache flush
wp option get permalink_structure
wp rewrite structure '/%postname%/'
wp rewrite flush
```

A normal flush refreshes database rules. To regenerate `.htaccess`, back it up and use `wp rewrite flush --hard`. For a single-site Apache installation, WP-CLI needs this in `wp-cli.yml`:

```yaml
apache_modules:
  - mod_rewrite
```

For Apache 404 errors, inspect MAMP's active Apache configuration (the usual path is shown):

```bash
rg -n 'rewrite_module|AllowOverride' /Applications/MAMP/conf/apache/httpd.conf
```

Ensure the module line is uncommented:

```apache
LoadModule rewrite_module modules/mod_rewrite.so
```

Check that the actual document root's `<Directory>` block permits `.htaccess` rewrite directives (`AllowOverride FileInfo` or an appropriate existing `AllowOverride All`). Restart MAMP after Apache changes, then test an inner-page URL.

## 10. WordPress core

Check first. Back up files and the database before running the update commands.

```bash
wp core version
wp core check-update
wp core verify-checksums
wp core update
wp core update-db
```

## 11. Inspect configuration

```bash
wp --info
wp config get DB_NAME
wp config get WP_DEBUG
```

Official command references: [WP-CLI](https://developer.wordpress.org/cli/commands/), [database check](https://developer.wordpress.org/cli/commands/db/check/), [database revision comparison](https://developer.wordpress.org/cli/commands/core/update-db/), [rewrite flush](https://developer.wordpress.org/cli/commands/rewrite/flush/).
