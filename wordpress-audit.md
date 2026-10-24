# Codex: WordPress Technical and Database Audit

Perform a comprehensive technical audit of this existing WordPress website, including its custom theme, PHP code, JavaScript, jQuery, plugins, dependencies, and MySQL database.

The goal is to identify compatibility problems, outdated code, security concerns, database issues, and opportunities to improve reliability and maintainability.

This is an existing website that must continue functioning as expected. Preserve its existing functionality and avoid unnecessary modernization.

**IMPORTANT: This is an audit-only task. Do not implement changes until I have reviewed and explicitly approved your recommendations.**

## 1. Project Review

Begin by inspecting the local repository and identifying:

- WordPress version, if available
- Current PHP version and compatibility requirements
- Active WordPress theme and any parent/child theme relationships
- Custom PHP functions, templates, hooks, and filters
- Installed plugins and their versions, where available
- JavaScript and jQuery dependencies
- Composer, npm, and other package dependencies
- Existing build tools, where relevant
- The local SQL database export and its structure

Establish the current state of the project before making recommendations.

If something cannot be determined from the repository, explain what additional information would be needed.

## 2. PHP and WordPress Compatibility

Review the custom theme and any available custom plugin code for:

- Deprecated or removed PHP functions
- PHP 8.x compatibility problems
- Deprecated WordPress functions, hooks, and APIs
- Incorrect or outdated WordPress API usage
- PHP warnings, notices, and potential fatal errors
- Code that may break with WordPress or PHP upgrades
- Incorrect assumptions about PHP data types or return values
- Error handling and reliability concerns
- Inefficient or unnecessary database queries
- Duplicated or unnecessarily complicated PHP functionality

Identify which PHP and WordPress versions the code currently appears compatible with.

Recommend a practical upgrade path if compatibility problems exist. Do not assume the newest available version is appropriate without evaluating dependencies.

Distinguish between changes that are necessary for compatibility and optional code improvements.

Do not recommend rewriting working code solely because it uses an older but supported approach.

## 3. JavaScript and jQuery

Review the theme's JavaScript and jQuery implementation for:

- Deprecated jQuery methods and APIs
- Outdated or unsupported JavaScript libraries
- Compatibility with WordPress-bundled jQuery
- Potential JavaScript errors or broken functionality
- Duplicate or conflicting dependencies
- Incorrect script registration and enqueueing
- Improper dependency declarations or load order
- Unnecessary reliance on jQuery
- Opportunities to simplify code using vanilla JavaScript
- Outdated build tools and package dependencies
- Potential incompatibilities with future WordPress updates

Recommend replacing jQuery only where doing so offers a clear benefit.

Avoid unnecessary rewrites or introducing new frameworks.

Preserve existing behavior and functionality.

## 4. Plugin and Dependency Compatibility

Review available plugin information, custom integrations, and dependency manifests.

Identify:

- Outdated plugin versions
- Plugins that may no longer be maintained
- Known compatibility concerns with newer PHP or WordPress versions
- Deprecated APIs used by custom plugins or integrations
- Potential conflicts between plugins
- Unnecessary or duplicated dependencies
- Dependencies that could prevent PHP or WordPress upgrades
- Custom functionality dependent on legacy plugins

Distinguish between installed plugins, active plugins, and dependencies merely referenced in the code.

Do not assume a plugin is unnecessary simply because its functionality is not obvious from the theme.

Do not update, deactivate, or remove anything.

## 5. Security and Code Maintainability

Inspect the custom PHP and JavaScript for:

- Missing input validation and sanitization
- Missing output escaping
- SQL injection vulnerabilities
- Cross-site scripting vulnerabilities
- Unsafe AJAX or REST API handlers
- Missing nonce verification
- Missing user capability checks
- Hardcoded credentials or sensitive information
- Insecure file handling or uploads
- Unsafe external API integrations
- Deprecated or potentially vulnerable dependencies
- Duplicate functions and unnecessary code
- Poorly organized custom functionality
- Custom implementations that could use supported WordPress APIs

Identify confirmed security issues separately from potential vulnerabilities requiring further investigation.

Do not expose sensitive values in the audit report.

## 6. MySQL Database Audit

The local repository contains a `.sql` file exported from the WordPress database.

Inspect this file as part of the audit. Do not connect to the local MySQL server unless I explicitly approve it.

### Database Structure and Compatibility

Review:

- WordPress core tables
- Custom tables
- Plugin-created tables
- Table structures and column definitions
- Primary keys and indexes
- Character sets and collations
- Deprecated database patterns
- Potential compatibility issues with current supported MySQL versions
- Potential issues with future WordPress upgrades

Pay particular attention to custom tables and plugin-generated structures.

Avoid recommending direct modifications to standard WordPress core tables unless a confirmed problem requires investigation.

### Database Performance and Optimization

Look for:

- Excessively large autoloaded options
- Potentially inefficient custom tables or indexes
- Unnecessary or redundant indexes
- Large amounts of transient data
- Excessive post revisions
- Potentially unused plugin tables
- Unnecessary database records
- Storage patterns that may affect performance
- Inefficient database usage in custom PHP code

Do not assume that large tables or large amounts of data automatically indicate a problem.

Consider how WordPress and its plugins use the data before recommending cleanup.

### Data Integrity

Look for potential issues involving:

- Orphaned post metadata
- Orphaned user metadata
- Orphaned term relationships
- Inconsistent data relationships
- Invalid or unexpected references
- Duplicate records where uniqueness is expected
- Custom tables with potential integrity problems
- Serialized data that may have been damaged during previous migrations

Distinguish between problems that can be confirmed from the SQL export and those requiring additional database queries.

### Plugin and Legacy Data

Identify:

- Tables potentially associated with removed plugins
- Legacy plugin settings
- Unused options
- Old migration artifacts
- Potentially abandoned custom tables
- Data structures associated with outdated functionality

Do not recommend deleting tables or records without identifying what created them and whether they are still required.

### Database Security

Check for:

- Sensitive configuration values stored unnecessarily
- Potentially exposed credentials or API keys
- Insecure data storage patterns
- Legacy authentication-related structures
- Sensitive information that should not be committed to version control

Do not reproduce passwords, password hashes, API keys, tokens, personal information, or other sensitive values in the report.

### Database Recommendations

For each finding, explain:

1. Which table or structure is affected.
2. What the issue is.
3. Why it matters.
4. Whether the issue is confirmed or requires additional investigation.
5. What improvement is recommended.
6. Any potential risks to WordPress
