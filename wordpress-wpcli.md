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

Official documentation: https://wp-cli.org/

## 2. Use MAMP's PHP

WP-CLI needs to use a PHP version compatible with the local WordPress installation.

List available MAMP PHP versions:

```bash
ls /Applications/MAMP/bin/php/
```

Add the selected version to your terminal's PATH:

```bash
export PATH="/Applications/MAMP/bin/php/<php-version>/bin:$PATH"
```

Verify:

```bash
which php
php -v
wp --info
```

To make the PHP PATH change persistent, add the `export` command to `~/.zshrc`.

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

```bash
# Verify connection
wp db check

# Export database
wp db export backup.sql

# Import database (overwrites matching tables)
wp db import backup.sql

# Open MySQL prompt
wp db cli

# Optimize tables
wp db optimize
```

Back up the database before importing or making significant changes.

## 9. Cache and Permalinks

```bash
# Flush WordPress object cache
wp cache flush

# Flush rewrite rules
wp rewrite flush

# Display permalink structure
wp option get permalink_structure
```

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

# Check WordPress configuration
wp config list

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
