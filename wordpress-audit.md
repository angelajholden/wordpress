# WordPress Technical and Database Audit

Use this checklist to audit an existing WordPress installation. Inspect and report only; do not implement changes until recommendations have been reviewed and explicitly approved. Preserve working behavior and avoid unnecessary modernization.

## Plugin restriction — audits and future implementation

**Codex must NEVER modify WordPress plugin source files.** This applies to `wp-content/plugins/`, must-use plugins, and every other installed plugin directory, during both audits and future implementation work.

- Plugin-file inspection is strictly read-only.
- Do not edit, patch, rewrite, replace, or delete plugin files, including PHP, JavaScript, CSS, and configuration files.
- Do not install, update, activate, deactivate, or remove plugins.
- Do not execute WP-CLI plugin-management commands, including `wp plugin list`.
- Do not create custom patches for third-party plugins or alter installed plugins to resolve PHP warnings or errors.
- If a problem is found, document it and recommend an officially supported solution. Plugin actions require separate explicit user approval; approval of other audit recommendations does not authorize them. Never treat implementation approval as permission to modify plugin source.

The commands in [wordpress-wpcli.md](wordpress-wpcli.md#5-plugins--manual-reference-only) are manual references, not authorization for Codex to execute them.

## 1. Establish the project baseline

Inspect available files and record:

- WordPress and PHP versions, distinguishing known versions from compatibility estimates.
- Active theme and parent/child relationship, where confirmed.
- Custom PHP functions, templates, hooks, and integrations.
- Installed plugins and available version metadata, using read-only file inspection; do not infer activation from directory presence.
- JavaScript, jQuery, Composer/npm dependencies, and existing build tools.
- Available SQL exports and their database structures.

Identify missing information rather than assuming it. Do not run package scripts or make changes during inspection.

## 2. Review PHP and WordPress compatibility

Inspect custom code for deprecated/removed PHP features, outdated WordPress APIs, fatal errors, type assumptions, error handling, and inefficient queries. Apply the read-only restriction to any plugin code.

Record the affected file/function, likely runtime impact, and compatible PHP/WordPress versions where supported by evidence. Separate necessary compatibility fixes from optional improvements. Do not recommend rewriting working code solely because it is old.

## 3. Review JavaScript and jQuery

Check deprecated APIs, bundled WordPress jQuery compatibility, duplicate dependencies, asset enqueue order, browser errors, and outdated build dependencies. Recommend vanilla JavaScript only where it offers a clear benefit; preserve behavior and avoid new frameworks.

## 4. Review plugins and dependencies

Inspect metadata and source read-only for maintenance status, compatibility concerns, deprecated APIs, security issues, conflicts, and dependencies that block upgrades. Distinguish confirmed use from files merely present on disk.

Do not assume an unfamiliar plugin is unnecessary. Document supported upgrade/replacement options for separate review; follow the mandatory plugin restriction above.

## 5. Review security and maintainability

Check custom PHP/JavaScript for:

- Input validation, sanitization, and output escaping.
- SQL injection, XSS, unsafe AJAX/REST handlers, nonce checks, and capability checks.
- Hardcoded secrets, unsafe file handling, and external integrations.
- Duplicated code, unreliable error handling, and unnecessary custom implementations of WordPress APIs.

Separate confirmed vulnerabilities from concerns requiring investigation. Never reproduce credentials, hashes, tokens, or private user data in the report.

## 6. Inspect the database snapshot

If an SQL export is available, inspect it without importing it. Do not connect to local or production MySQL without explicit approval. A snapshot may differ from the running database; label conclusions accordingly.

Review:

- Core, custom, and plugin tables; column types, keys, indexes, character sets, and collations.
- Schema compatibility with the installed WordPress/MySQL versions.
- Serialized data and possible migration damage.
- Potential orphaned metadata, inconsistent references, duplicate records, and broken relationships.
- Large autoloaded options, transient/revision growth, and inefficient queries supported by code evidence.
- Legacy plugin tables/options and sensitive data that should not be stored or committed.

Do not equate size with a performance defect. Do not propose deleting unfamiliar tables or records without confirming ownership and use. Avoid changes to core tables unless a confirmed mismatch requires investigation; a recorded WordPress database revision alone does not prove schema correctness.

If live checks are needed, request approval and refer to the [database inspection procedure](wordpress-wpcli.md#8-check-the-database). Do not execute it as part of a file-only audit.

## 7. Report and prioritize

For each finding, provide:

1. Affected file, function, dependency, table, or column.
2. Evidence and whether the finding is confirmed or needs verification.
3. Practical impact and priority.
4. Recommended action, compatibility requirements, and risks.
5. Validation steps and backup/rollback needs for any future change.

Keep plugin findings read-only and recommend officially supported solutions for separate approval. Do not implement recommendations, execute maintenance commands, or change data during the audit.
