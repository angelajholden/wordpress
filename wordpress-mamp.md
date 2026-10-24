# WordPress → MAMP

Replace `<placeholders>`. Start MAMP with `/Applications/MAMP/htdocs` as its document root; note its Apache/MySQL ports and database credentials. Have WP-CLI available with MAMP's PHP. Keep the downloaded files and SQL as backups.

## 1. Download WordPress files — local terminal

Copy the entire installation, including hidden files and `wp-content`.

```bash
mkdir -p ~/Downloads/<site>
rsync -raz --progress <ssh-user>@<server>:/<wordpress-path>/ ~/Downloads/<site>/
```

## 2. Export the database — server terminal

Connect from your Mac; use the database settings from the server's `wp-config.php`. Run `mysqldump` at the shell, not the MySQL prompt; `-p` prompts for the password.

```bash
ssh <ssh-user>@<server>
cd /<wordpress-path>
mysqldump -u <production-db-user> -p <production-db-name> > /<private-backup-path>/<site>.sql
exit
```

## 3. Download the SQL file — local terminal

```bash
scp <ssh-user>@<server>:/<private-backup-path>/<site>.sql ~/Downloads/
ls -lh ~/Downloads/<site>.sql
```

## 4. Copy WordPress into MAMP — local terminal

```bash
mkdir -p /Applications/MAMP/htdocs/<site>
rsync -raz --progress ~/Downloads/<site>/ /Applications/MAMP/htdocs/<site>/
```

## 5. Open MAMP's MySQL client — local terminal

Enter the actual MAMP database password when prompted.

```bash
cd /Applications/MAMP/Library/bin
./mysql -h 127.0.0.1 -P <mysql-port> -u <local-db-user> -p
```

## 6. Create the database — MySQL prompt

```sql
CREATE DATABASE `<local-db-name>` CHARACTER SET utf8mb4;
```

## 7. Import the SQL file — MySQL prompt

Use a new, empty database; importing over existing tables can overwrite them. Use the SQL file's absolute Mac path.

```sql
USE `<local-db-name>`;
source /Users/<mac-user>/Downloads/<site>.sql
SHOW TABLES;
EXIT;
```

## 8. Update `wp-config.php` — local file

Edit `/Applications/MAMP/htdocs/<site>/wp-config.php`; replace existing definitions and preserve `$table_prefix`.

```php
define( 'DB_NAME', '<local-db-name>' );
define( 'DB_USER', '<local-db-user>' );
define( 'DB_PASSWORD', '<local-db-password>' );
define( 'DB_HOST', '127.0.0.1:<mysql-port>' );
```

## 9. Replace URLs — local terminal

The original SQL is your backup. Confirm the database settings point locally before running these commands. Use the actual production URL; update any `WP_HOME`/`WP_SITEURL` constants locally too.

```bash
cd /Applications/MAMP/htdocs/<site>
wp option get home
wp option get siteurl
wp search-replace 'https://<production-domain>' 'http://localhost:<apache-port>/<site>' --all-tables-with-prefix --skip-columns=guid --dry-run
wp search-replace 'https://<production-domain>' 'http://localhost:<apache-port>/<site>' --all-tables-with-prefix --skip-columns=guid
wp rewrite flush
```

Review the dry run first. WP-CLI preserves serialized data; avoid the notes' `sed`/SQL `REPLACE()` migration commands, which can corrupt serialized values.

## 10. Test locally — browser

Open `http://localhost:<apache-port>/<site>/` and `/wp-admin/`. Check login, pages, images, and permalinks; save **Settings → Permalinks** if needed. Do not run the WordPress installer for an imported site.
