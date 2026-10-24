# WordPress → MAMP

My backup and local restore snippets for WordPress sites hosted on DigitalOcean. Local setup: MAMP, Apache port **80**, MySQL port **3306**, WordPress files in `/Applications/MAMP/htdocs/`.

The commands below use `/var/www/html` for a typical server and `angelajholden` as a local example. Change site names, remote login details, and database credentials as needed.

## Back up the remote site

### 1. Connect to the server — Mac terminal

```bash
ssh user@123.123.123.123
```

Use the site's SSH user and droplet IP. SSH connects **from the Mac to the remote server**; it isn't needed to work on a site already downloaded.

### 2. Export the database — remote server

Keep database dumps **outside the web root**, not in `wp-content`.

```bash
sudo mkdir -p /var/www/backups
sudo chown "$(whoami):$(id -gn)" /var/www/backups
chmod 700 /var/www/backups
mysqldump -u wordpress -p wordpress_db > /var/www/backups/wordpress.sql
```

Here, `wordpress` and `wordpress_db` are example MySQL user/database names; use the values in the site's `wp-config.php`. The `-p` flag prompts for the password. Run `mysqldump` at the **shell**, not inside MySQL. Confirm it finished successfully before archiving.

### 3. ZIP `wp-content`, `wp-config.php`, and SQL — remote server

```bash
cd /var/www
zip -r backups/wordpress-backup.zip html/wp-content html/wp-config.php backups/wordpress.sql
unzip -l backups/wordpress-backup.zip
```

Everything goes into **one ZIP**, but neither the SQL nor ZIP is placed in the public website directory. Keep the ZIP private: it includes credentials and database contents.

If the site uses custom files outside `wp-content`, include those separately.

### 4. Check for old forgotten backups — remote server

```bash
find /var/www/html/wp-content -type f \( -name '*.sql' -o -name '*.sql.gz' -o -name '*.zip' -o -name '*.tar.gz' \)
```

Review the results and remove abandoned backups manually. Don't blindly delete files owned by plugins.

### 5. Download the ZIP — Mac terminal

Preferred method: **SCP directly to the server IP**, even if the domain's A record is gone.

```bash
scp user@123.123.123.123:/var/www/backups/wordpress-backup.zip ~/Downloads/
unzip -t ~/Downloads/wordpress-backup.zip
```

Alternative if the archive is intentionally available through a URL: `wget https://example.com/backup.zip`. **Don't remove the domain's A record before downloading and verifying your backup.** SCP is preferable because it doesn't require a public download URL or working domain DNS.

You can also download a site that's already backed up without repeating the export.

## Restore locally in MAMP

### 6. Install fresh WordPress core — Mac

Download a fresh, compatible WordPress release from [wordpress.org](https://wordpress.org/download/) and extract it to:

```text
/Applications/MAMP/htdocs/angelajholden
```

Start MAMP. Use the newly downloaded WordPress core rather than copying old core files from production. Confirm the version is compatible with the site's theme, plugins, and database before upgrading.

### 7. Extract the backup and restore `wp-content` — Mac terminal

```bash
mkdir -p ~/Downloads/wordpress-restore
unzip ~/Downloads/wordpress-backup.zip -d ~/Downloads/wordpress-restore
rsync -a ~/Downloads/wordpress-restore/html/wp-content/ /Applications/MAMP/htdocs/angelajholden/wp-content/
```

The extracted archive also contains `html/wp-config.php` and `backups/wordpress.sql`. Keep the original ZIP as a backup.

### One-time terminal PATH setup — Mac

Add MAMP's PHP and MySQL tools to your shell so `wp`, `mysql`, and `mysqlcheck` work in new Terminal windows. This example uses **PHP 8.4.1** and **MySQL 8.0**; check the versions installed in your MAMP first.

```bash
ls /Applications/MAMP/bin/php/
ls /Applications/MAMP/Library/bin/
nano ~/.zshrc
```

Add these lines to `~/.zshrc`:

```bash
export PATH="/Applications/MAMP/bin/php/php8.4.1/bin:$PATH"
export PATH="/Applications/MAMP/Library/bin/mysql80/bin:$PATH"
```

Save, reload, and verify:

```bash
source ~/.zshrc
which php
which mysql
which mysqlcheck
php -v
wp --info
```

> [!NOTE]
> This is **optional, one-time setup**. These lines make MAMP's PHP/MySQL tools the defaults in new Terminal sessions, including outside WordPress projects. Remove or adjust them when switching versions or using another environment. For only the current session, run the two `export PATH` commands in Terminal without editing `~/.zshrc`.

> [!TIP]
> If `wp db check` reports `env: mysqlcheck: No such file or directory`, verify `which mysqlcheck` and your PATH. If MySQL reports `ERROR 2003` / connection refused, start MAMP's MySQL server and verify port **3306** before changing passwords.

### 8. Connect to MAMP MySQL — Mac terminal

```bash
/Applications/MAMP/Library/bin/mysql80/bin/mysql -h 127.0.0.1 -P 3306 -u root -p
```

Enter your MAMP MySQL password when prompted.

### 9. Create and import the database — MySQL prompt

```sql
CREATE DATABASE angelajholden CHARACTER SET utf8mb4;
USE angelajholden;
source /Users/angelajholden/Downloads/wordpress-restore/backups/wordpress.sql
SHOW TABLES;
EXIT;
```

The `source` path uses your Mac's download folder. Import into a new, empty database; change the database name if it already exists.

### 10. Configure `wp-config.php` — Mac

Use the saved `html/wp-config.php` as a reference (especially for **multisite** settings). Copy it into the local WordPress root if appropriate, then change its database settings:

```php
define( 'DB_NAME', 'angelajholden' );
define( 'DB_USER', 'root' );
define( 'DB_PASSWORD', 'your-local-mamp-password' );
define( 'DB_HOST', '127.0.0.1:3306' );
```

Keep the correct `$table_prefix`. Review custom constants, salts, and multisite network settings; don't reuse production-only URLs or settings without checking them. Never commit `wp-config.php`.

### 11. Replace URLs — Mac terminal

Use WP-CLI with MAMP's PHP; see [wordpress-wpcli.md](wordpress-wpcli.md). Replace the **actual previous site URL** below:

```bash
cd /Applications/MAMP/htdocs/angelajholden
wp option get home
wp option get siteurl
wp search-replace 'https://example.com' 'http://localhost/angelajholden' --all-tables-with-prefix --skip-columns=guid --dry-run
wp search-replace 'https://example.com' 'http://localhost/angelajholden' --all-tables-with-prefix --skip-columns=guid
wp rewrite flush
wp db check
```

Review the dry run first. WP-CLI handles serialized data correctly; don't use `sed` or plain SQL `REPLACE()` for WordPress URL migrations. Check any `WP_HOME` / `WP_SITEURL` constants in `wp-config.php`. **Multisite URL migrations require additional network-specific checks.**

### 12. Test locally

Open `http://localhost/angelajholden/` and `http://localhost/angelajholden/wp-admin/`. Check login, pages, media, and permalinks. If inner pages return Apache 404 errors, follow the Apache rewrite steps below.

Don't run the WordPress installer for an imported site. If WordPress asks for a database update after changing core versions, use its normal database upgrade process.

### Apache mod_rewrite — fix permalink 404s (MAMP)

If the home page works but URLs like `/blog/` show a plain **Apache 404**, check MAMP's rewrite module.

**Mac terminal:**

```bash
grep -n 'rewrite_module' /Applications/MAMP/conf/apache/httpd.conf
nano /Applications/MAMP/conf/apache/httpd.conf
```

Find this line and remove the leading `#` if present:

```apache
LoadModule rewrite_module modules/mod_rewrite.so
```

Save the file, **stop and restart MAMP's servers**, then test an inner page again. This was the fix during the stream.

If pages still 404, check that Apache allows `.htaccess` overrides for `/Applications/MAMP/htdocs` (`AllowOverride All` or the appropriate rewrite permissions). Also verify that the site's `.htaccess` contains the correct WordPress rewrite rules.

**Final reminder:** Verify the downloaded ZIP before changing DNS, deleting the droplet, or removing the original backup. Keep old backups private and check `wp-content` for abandoned SQL/ZIP files.
