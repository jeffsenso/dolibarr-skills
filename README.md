# .ai — Copilot Skills & Customizations

This folder contains GitHub Copilot skills for this Dolibarr development workspace.

## Structure

```
.ai/
├── README.md           ← this file
├── CHANGELOG.md        ← history of skills added/changed
└── skills/
    └── dolibarr-development/
        ├── SKILL.md                        ← main skill entry point
        └── references/
            ├── module-structure.md         ← module file tree, descriptor, menus, tabs, permissions, boxes
            ├── hooks-triggers.md           ← hooks system, trigger classes, common hook methods & events
            ├── coding-rules.md             ← PHP, SQL, HTML norms (PSR-12, GETPOST, dates, amounts)
            └── technical-components.md     ← DAO pattern, tabs, translations, config, forms, API, cron
```

## Skills

### dolibarr-development
A comprehensive skill for Dolibarr ERP/CRM development — covers both core contributors and external module developers.

**Invoke by**: asking Copilot anything about Dolibarr development, module creation, hooks, triggers, coding rules, SQL tables, PHP pages, menus, tabs, permissions, extrafields, PDF templates, cron jobs, REST API.

**What it covers:**
- Module creation workflow (Module Builder, descriptor, SQL, DAO, pages)
- Hooks system: implement `actions_mymodule.class.php`, declare contexts, return codes, common hook methods
- Triggers: react to business events (BILL_CREATE, ORDER_VALIDATE, etc.)
- Coding rules: PSR-12 PHP, SQL norms (no `SELECT *`, no SQL date functions, soft FK), HTML/CSS
- Technical components: DAO Active Record pattern, MVC structure, tabs, menus, permissions, translations, constants, forms, extrafields, cron, REST API

**Source**: https://wiki.dolibarr.org/index.php/Developer_documentation
