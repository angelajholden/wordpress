# WP-CLI — Local WordPress Development with MAMP

Common WP-CLI commands for managing WordPress installations locally on macOS with MAMP.

## 1. Install WP-CLI

Using Homebrew:

```bash
brew install wp-cli
```

Verify installation:

```bash
wp --info
```

Official documentation: [WP-CLI](https://wp-cli.org/).

## 2. Use MAMP's PHP and MySQL

Homebrew's PHP may differ from MAMP's web PHP. In the stream, WP-CLI initially used Homebrew PHP 8.5 and reported a bundled dependency deprecation warning; switching to MAMP PHP 8.4.1 removed it. Match the PHP selected in MAMP rather than treating the warning as a theme error.

**Local terminal:** identify installed versions, then prepend their executable directories. These paths worked on October 18, 2026; substitute your installed versions.

```bash
ls /Applications/MAMP/bin/php/
find /Applications/MAMP/Library -type f -name mysql
export PATH="/Applications/MAMP/bin/php/php8.4.1/bin:$PATH"
export PATH="/Applications/MAMP/Library/bin/mysql80/bin:$PATH"
which php
php -v
which mysql
mysql --version
wp --info
```

Confirm `wp --info` shows MAMP's PHP binary/version and MySQL binary. The MySQL directory also supplies the tools needed by database commands. These exports affect only the current terminal session; add them to `~/.zshrc` only if you intentionally want them to persist. The MySQL client version is separate from the running database server version.

## 3. Navigate to WordPress

Run WP-CLI commands from the WordPress installation directory:

```bash
cd /Applications/MAMP/htdocs/<site>
```

Verify WordPress is installed:

```bash
wp core is-installed
wp core version
```

## 4. WordPress URLs

Display the current URLs:

```bash
wp option get home
wp option get siteurl
```

Update URLs:

```bash
wp option update home 'http://localhost/<site>'
wp option update siteurl 'http://localhost/<site>'
```

Replace production URLs throughout the database.

Always run a dry run first:

```bash
wp search-replace 'https://example.com' 'http://localhost/<site>' --all-tables-with-prefix --skip-columns=guid --dry-run
```

Execute after reviewing the results:

```bash
wp search-replace 'https://example.com' 'http://localhost/<site>' --all-tables-with-prefix --skip-columns=guid
```

WP-CLI handles serialized WordPress data.

Use the actual MAMP port in local URLs if Apache isn't running on port 80.

## 5. Plugins

```bash
# List installed plugins
wp plugin list

# Check plugin updates
wp plugin list --update=available

# Update one plugin
wp plugin update <plugin-name>

# Update all plugins
wp plugin update --all

# Activate / deactivate
wp plugin activate <plugin-name>
wp plugin deactivate <plugin-name>

# Deactivate all plugins
wp plugin deactivate --all
```

## 6. Themes

```bash
# List installed themes
wp theme list

# Check theme updates
wp theme list --update=available

# Activate a theme
wp theme activate <theme-name>

# Update a theme
wp theme update <theme-name>
```

## 7. Users

```bash
# List users
wp user list

# Display user details
wp user get <username>

# Update a password
wp user update <username> --prompt=user_pass

# List administrators
wp user list --role=administrator
```

## 8. Database

**Local terminal, from the WordPress installation directory:** these commands use the database configured in `wp-config.php`.

```bash
# Check MySQL table integrity (also confirms the connection)
wp db check

# Compare the recorded WordPress DB revision with installed core; no update
wp core update-db --dry-run

# Back up before changing data or schema
wp db export backup.sql

# Open the configured database's MySQL prompt
wp db cli
```

Move `backup.sql` outside the web root after exporting; keep it out of Git. The [integrity check](https://developer.wordpress.org/cli/commands/db/check/) does not validate the expected WordPress schema. The [database revision comparison](https://developer.wordpress.org/cli/commands/core/update-db/) does not independently prove every expected column or index exists either.

**Read-only SQL at the MySQL prompt:** replace `wp_` with the installation's actual table prefix.

```sql
SHOW COLUMNS FROM wp_options LIKE 'option_name';
SHOW INDEX FROM wp_posts WHERE Key_name = 'type_status_author';
SELECT COUNT(*) AS null_option_names FROM wp_options WHERE option_name IS NULL;
EXIT;
```

Inspect `option_name`'s `Null` value and the index's columns/order (`Seq_in_index`), not just its name. Compare with the installed WordPress version's expected schema before planning corrections.

In the stream, integrity/revision checks passed, but `option_name` still allowed NULL; `type_status_author` already existed. The NULL-record count was zero. After a local export, the column was corrected and checked again. This is a finding from that installation, not a blanket instruction to alter core tables.

An `.sql` export is a snapshot. Reading it does not change the running MySQL database, and inspecting or changing the running database does not update an old export. Export again after local changes before deployment. The [planned production cutover](wordpress-mamp.md#production-database-cutover--planned-workflow) is documented separately.

## 9. Cache and Permalinks

**Local terminal, from the WordPress installation directory:**

```bash
wp cache flush
wp option get permalink_structure
wp rewrite structure '/%postname%/'
wp rewrite flush
```

The structure command changes permalinks; use `/%postname%/` only if that is the intended format. A normal flush refreshes database rewrite rules; it does not regenerate `.htaccess`.

If `.htaccess` rules need rebuilding, back up the file first and use `wp rewrite flush --hard`. For a single-site Apache install, WP-CLI also needs the following in its `wp-cli.yml` (see the [flush reference](https://developer.wordpress.org/cli/commands/rewrite/flush/)):

```yaml
apache_modules:
  - mod_rewrite
```

Apache must load the module and read `.htaccess`. The stream's navigation returned Apache 404s even after changing/flushing WordPress permalinks. The rewrite module was commented out; enabling it and restarting MAMP made navigation work.

**Local terminal:** inspect MAMP's Apache configuration:

```bash
rg -n 'rewrite_module|AllowOverride' /Applications/MAMP/conf/apache/httpd.conf
```

Ensure this line is uncommented in the active configuration:

```apache
LoadModule rewrite_module modules/mod_rewrite.so
```

If rewrites still fail, check that the `<Directory>` block for the actual document root allows `.htaccess` rewrite directives (`AllowOverride FileInfo`, or the existing broader `AllowOverride All`). Restart MAMP after Apache configuration changes, then test an inner-page URL again.

## 10. WordPress Core

```bash
# Check WordPress version
wp core version

# Check for updates
wp core check-update

# Verify core files
wp core verify-checksums

# Update WordPress
wp core update

# Update WordPress database
wp core update-db
```

## 11. Troubleshooting

```bash
# Display WP-CLI environment
wp --info

# Check the configured database name without displaying passwords
wp config get DB_NAME

# List active plugins
wp plugin list --status=active

# Run a command without loading ordinary plugins/themes
wp --skip-plugins --skip-themes plugin list

# Inspect the WordPress debug setting
wp config get WP_DEBUG
```

**Notes**

- Run commands from the correct WordPress installation.
- Confirm MAMP's MySQL server is running before using database commands.
- Keep production databases and credentials separate from local installations.
- Back up before WordPress, plugin, or database updates.
- Use `wp search-replace` rather than raw SQL replacement when migrating WordPress URLs.
