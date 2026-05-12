# Changelog — .ai Skills

## [1.0.0] — 2026-05-12

### Added
- **skill: `dolibarr-development`** — Initial version

  Crawled and distilled from https://wiki.dolibarr.org/index.php/Developer_documentation

  Files created:
  - `skills/dolibarr-development/SKILL.md` — Main skill entry point with step-by-step module build guide, architecture overview, quick reference tables, and links to references
  - `skills/dolibarr-development/references/module-structure.md` — Full module file tree, `modMyModule.class.php` descriptor template, menus/tabs/permissions/boxes declaration examples, Module Builder usage, module ID registry link
  - `skills/dolibarr-development/references/hooks-triggers.md` — Hooks system theory (hooks vs triggers), step-by-step hook implementation, `actions_mymodule.class.php` template with common hook methods, return code table, trigger class template, common trigger event names, grep commands to find hooks/contexts in source
  - `skills/dolibarr-development/references/coding-rules.md` — PHP rules (PSR-12, GETPOST, global variables, dol_syslog, dol_include_once), SQL rules (table structure, field types, key naming, transaction pattern, forbidden constructs), HTML rules (MVC comment tags, CSS classes, JS guard)
  - `skills/dolibarr-development/references/technical-components.md` — main.inc.php bootstrap, DAO class pattern with full CRUD, tabs system with object-lib pairs, translation system, permission checks, config constants, Form helpers, URL/path constants, extrafields, error reporting, cron script pattern, REST API extension
