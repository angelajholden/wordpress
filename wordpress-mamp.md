# WordPress → MAMP

Replace placeholders consistently; see [conventions](README.md#conventions). Use compatible PHP/MySQL versions. Start MAMP with `/Applications/MAMP/htdocs` as its document root and note its actual Apache/MySQL ports and credentials. Configure [WP-CLI's local PATH](wordpress-wpcli.md#2-select-mamps-php-and-mysql) before using it.

## Local installation

### 1. Download files — local terminal

Copy the full installation, including hidden files and `wp-content`, into an empty destination. Keep this download as a backup.

```bash
mkdir -p ~/Downloads/<site>
rsync -raz --progress <ssh-user>@<server>:/<wordpress-path>/ ~/Downloads/<site>/
```

### 2. Export the database — local terminal, then remote terminal

Run SSH locally. Run `mysqldump` on the server using the existing `wp-config.php` database settings; `-p` prompts for the password. Add the configured host/port if the database is not on the default local connection. Store the export outside the web root and continue only after it succeeds.

```bash
ssh <ssh-user>@<server>
```

**Remote terminal:**

```bash
mysqldump -u <production-db-user> -p <production_db_name> > /<private-backup-path>/<site>.sql
exit
```

### 3. Download SQL and copy files into MAMP — local terminal

```bash
scp <ssh-user>@<server>:/<private-backup-path>/<site>.sql ~/Downloads/
ls -lh ~/Downloads/<site>.sql
mkdir -p /Applications/MAMP/htdocs/<site>
rsync -raz --progress ~/Downloads/<site>/ /Applications/MAMP/htdocs/<site>/
```

### 4. Connect to MAMP MySQL — local terminal

Locate the executable using the [PATH setup](wordpress-wpcli.md#2-select-mamps-php-and-mysql). For example, MySQL 8.0 may be at `/Applications/MAMP/Library/bin/mysql80/bin/mysql`; use your installed path. With that directory on PATH:

```bash
mysql -h 127.0.0.1 -P <mysql-port> -u <local-db-user> -p
```

### 5. Create and import — local MySQL prompt

Use a new empty database. Imports can overwrite existing tables; stop on errors. Keep the original SQL backup unchanged.

```sql
CREATE DATABASE <local_db_name> CHARACTER SET utf8mb4;
USE <local_db_name>;
source /Users/<mac-user>/Downloads/<site>.sql
SHOW TABLES;
EXIT;
```

### 6. Configure WordPress — local file edit

Edit `/Applications/MAMP/htdocs/<site>/wp-config.php`. Replace existing definitions and preserve the imported `$table_prefix`.

```php
define( 'DB_NAME', '<local_db_name>' );
define( 'DB_USER', '<local-db-user>' );
define( 'DB_PASSWORD', '<local-db-password>' );
define( 'DB_HOST', '127.0.0.1:<mysql-port>' );
```

### 7. Replace URLs — local terminal

Confirm the database connection is local. Update any `WP_HOME`/`WP_SITEURL` overrides in the local configuration too. Use the actual production URL, including `www` or an installation path. Preserve any intentional difference between `home` and `siteurl`.

```bash
cd /Applications/MAMP/htdocs/<site>
wp option get home
wp option get siteurl
wp search-replace 'https://<production-domain>' 'http://localhost:<apache-port>/<site>' --all-tables-with-prefix --skip-columns=guid --dry-run
wp search-replace 'https://<production-domain>' 'http://localhost:<apache-port>/<site>' --all-tables-with-prefix --skip-columns=guid
wp rewrite flush
```

Review the dry run before applying. WP-CLI preserves serialized data; plain SQL `REPLACE()` or `sed` can corrupt serialized string lengths. Leave GUIDs unchanged. At Apache port 80, the port suffix may be omitted.

### 8. Test — local browser and terminal

Open `http://localhost:<apache-port>/<site>/` and `/wp-admin/`. Check login, pages, media, theme rendering, and permalinks. Do not run the installer for a restored site. Use the [database checks](wordpress-wpcli.md#8-check-the-database) and [rewrite checks](wordpress-wpcli.md#9-check-cache-and-permalinks) when needed.

## Production database cutover

Finish local development and testing first. This replaces production content: account for new posts, users, orders, and other writes before making the local database authoritative. Prepare the new database ahead of time if useful, then block visitor/background writes during cutover and testing. Ensure deployed WordPress files are compatible with the updated database.

### 1. Export and upload — local terminal

```bash
cd /Applications/MAMP/htdocs/<site>
wp db export /Users/<mac-user>/Downloads/<site>-updated.sql
scp /Users/<mac-user>/Downloads/<site>-updated.sql <ssh-user>@<server>:/<private-backup-path>/
ssh <ssh-user>@<server>
```

### 2. Back up production — remote terminal

Use the current connection settings. Confirm the export succeeds and retain the old database and credentials privately for rollback.

```bash
mysqldump -u <production-db-user> -p <production_db_name> > /<private-backup-path>/production-before-cutover.sql
mysql -u <mysql-admin-user> -p
```

### 3. Create a new database/user and import — remote MySQL prompt

Match the user host to the production connection; this example uses `localhost`. Confirm the SQL contains no `USE` or qualified database names that would redirect the import elsewhere. Stop on import errors.

```sql
CREATE DATABASE <new_db_name> CHARACTER SET utf8mb4;
CREATE USER '<new-db-user>'@'localhost' IDENTIFIED BY '<new-db-password>';
GRANT ALL PRIVILEGES ON <new_db_name>.* TO '<new-db-user>'@'localhost';
USE <new_db_name>;
source /<private-backup-path>/<site>-updated.sql
SHOW TABLES;
EXIT;
```

### 4. Switch the connection — remote file edit

During the maintenance window, update production `wp-config.php` with the new `DB_NAME`, `DB_USER`, and `DB_PASSWORD`. Keep the correct production `DB_HOST` and imported table prefix. Set any `WP_HOME`/`WP_SITEURL` overrides to production.

### 5. Check and replace URLs — remote terminal

WP-CLI now uses the newly configured database. Confirm the target before writing. Keep visitors blocked until replacement and testing finish; the imported site may initially redirect to localhost.

```bash
cd /<wordpress-path>
wp config get DB_NAME
wp db check
wp search-replace 'http://localhost:<apache-port>/<site>' 'https://<production-domain>' --all-tables-with-prefix --skip-columns=guid --dry-run
wp search-replace 'http://localhost:<apache-port>/<site>' 'https://<production-domain>' --all-tables-with-prefix --skip-columns=guid
wp rewrite flush
```

Use the exact local URL stored in the database, omitting the port suffix if it was omitted locally. Review the dry run before applying.

### 6. Test and retain rollback — production browser

Test the home page, admin login, media, and inner-page permalinks before reopening traffic. Retain the original database and connection details. Restoring that connection is a rollback option only while the original database remains intact; reconcile any post-cutover writes before switching back.
