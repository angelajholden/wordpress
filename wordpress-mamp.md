# WordPress → MAMP

Replace `<placeholders>`. Start MAMP with `/Applications/MAMP/htdocs` as its document root; note its Apache/MySQL ports and database credentials. Configure WP-CLI with MAMP's PHP/MySQL using [wordpress-wpcli.md](wordpress-wpcli.md#2-use-mamps-php-and-mysql). Keep the downloaded files and SQL as backups.

Stream example (October 18, 2026): Apache **80**, MySQL **3306**, PHP **8.4.1**, MySQL **8.0 client**, and MAMP's local **root** account. These are examples, not universal defaults; the client version does not identify the running server version. At Apache port 80, use `http://localhost/<site>` without a port suffix.

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

Locate the installed executable; `/Applications/MAMP/Library/bin/mysql` did not exist in the stream installation. Use the path matching your selected MySQL version. This MySQL 8.0 path worked; enter your actual local password when prompted.

```bash
find /Applications/MAMP/Library -type f -name mysql
/Applications/MAMP/Library/bin/mysql80/bin/mysql -h 127.0.0.1 -P <mysql-port> -u <local-db-user> -p
```

## 6. Create the database — MySQL prompt

Use a simple database name with letters, numbers, and underscores.

```sql
CREATE DATABASE <local_db_name> CHARACTER SET utf8mb4;
SHOW DATABASES;
```

## 7. Import the SQL file — MySQL prompt

Use a new, empty database; importing over existing tables can overwrite them. Use the SQL file's absolute Mac path.

```sql
USE <local_db_name>;
source /Users/<mac-user>/Downloads/<site>.sql
SHOW TABLES;
EXIT;
```

## 8. Update `wp-config.php` — local file

Edit `/Applications/MAMP/htdocs/<site>/wp-config.php`; replace existing definitions and preserve `$table_prefix`.

```php
define( 'DB_NAME', '<local_db_name>' );
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

Open `http://localhost:<apache-port>/<site>/` and `/wp-admin/`. Check login, pages, images, and permalinks. For Apache 404s, use the short [permalink/rewrite checks](wordpress-wpcli.md#9-cache-and-permalinks). Run the [database checks](wordpress-wpcli.md#8-database) before development. Do not run the WordPress installer for an imported site.

## Production database cutover — planned workflow

Discussed during the stream; **not deployed or tested there**. Finish development/database updates locally and test first. This replaces production content: account for new posts, users, orders, and other writes before making local data authoritative. Use a controlled maintenance window that blocks visitor/background writes through cutover and testing; the new database can be prepared beforehand. Keep deployed WordPress files compatible with the local database.

### 1. Export locally and upload — local terminal

```bash
cd /Applications/MAMP/htdocs/<site>
wp db export /Users/<mac-user>/Downloads/<site>-updated.sql
scp /Users/<mac-user>/Downloads/<site>-updated.sql <ssh-user>@<server>:/<private-backup-path>/
```

### 2. Back up production and prepare a new database — server terminal

Use the current production connection settings for the backup; confirm the export succeeds before continuing. Preserve the old database and connection details privately for rollback.

```bash
ssh <ssh-user>@<server>
mysqldump -u <current-db-user> -p <current_db_name> > /<private-backup-path>/production-before-cutover.sql
mysql -u <mysql-admin-user> -p
```

**MySQL prompt:** use a new database/user name and password. Match the user host to the production database connection (this example uses `localhost`).

```sql
CREATE DATABASE <new_db_name> CHARACTER SET utf8mb4;
CREATE USER '<new-db-user>'@'localhost' IDENTIFIED BY '<new-password>';
GRANT ALL PRIVILEGES ON <new_db_name>.* TO '<new-db-user>'@'localhost';
USE <new_db_name>;
source /<private-backup-path>/<site>-updated.sql
SHOW TABLES;
EXIT;
```

Import only into the new empty database; stop on errors. Check the SQL has no `USE` or qualified database names that redirect the import elsewhere.

### 3. Switch the connection — production `wp-config.php`

During the maintenance window, replace `DB_NAME`, `DB_USER`, and `DB_PASSWORD` with the new connection details. Keep the correct production `DB_HOST` and imported table prefix. Update any `WP_HOME`/`WP_SITEURL` overrides to production too.

### 4. Check, replace URLs, and test — server terminal

After the config change, WP-CLI targets the **newly configured database**. Confirm that target before writing. Until URL replacement finishes, the imported site may redirect to localhost; keep visitors blocked.

```bash
cd /<wordpress-path>
wp config get DB_NAME
wp db check
wp search-replace 'http://localhost/<site>' 'https://<production-domain>' --all-tables-with-prefix --skip-columns=guid --dry-run
wp search-replace 'http://localhost/<site>' 'https://<production-domain>' --all-tables-with-prefix --skip-columns=guid
wp rewrite flush
```

Use the exact local URL, including a nonstandard port if applicable. Review the dry run before applying; WP-CLI preserves serialized data. Test the production home page, admin login, media, and inner-page permalinks before reopening traffic.

Retain the original database and saved connection details. Restoring that connection is a rollback option only while the original database is intact; account for any post-cutover writes before switching back.
