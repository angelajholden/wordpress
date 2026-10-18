# WordPress Development

A collection of notes, commands, and reference guides for managing, migrating, troubleshooting, and maintaining WordPress websites.

These guides document my personal workflow using macOS, MAMP, WP-CLI, and DigitalOcean.

## Documentation

### [WordPress + MAMP](wordpress-mamp.md)

Step-by-step commands for migrating an existing WordPress website from a DigitalOcean LAMP server to a local MAMP installation.

Includes downloading files, exporting and importing MySQL databases, updating configuration files, and replacing production URLs.

### [WordPress + WP-CLI](wordpress-wpcli.md)

Common WP-CLI commands for managing WordPress installations locally.

Includes WordPress core, plugins, themes, users, database operations, URL updates, and troubleshooting.

## WordPress Maintenance

My general workflow for maintaining existing WordPress websites:

1. Download the website and database to a local MAMP environment.
2. Use WP-CLI to inspect and manage the installation.
3. Audit PHP, WordPress compatibility, JavaScript, jQuery, plugins, and database structure.
4. Review and prioritize recommended improvements.
5. Implement approved changes incrementally and test locally.
6. Deploy tested changes to production.

## Environment

- **Operating system:** macOS
- **Local development:** MAMP
- **Database:** MySQL
- **Command-line tools:** WP-CLI, SSH, rsync, scp, mysqldump
- **Hosting:** DigitalOcean (Ubuntu/LAMP)

## Notes

These documents are intended as personal technical references rather than comprehensive tutorials. Commands may require adjustments depending on the WordPress installation, PHP version, or server configuration.

Production credentials, database exports, and client-specific information should not be committed to this repository.
