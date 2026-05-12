---
name: dolibarr-development
description: 'Dolibarr ERP/CRM developer skill. Use for: building or extending Dolibarr modules (external/custom), implementing hooks, triggers, DAO classes, SQL tables, menus, permissions, tabs, boxes, PDF templates, cron scripts. Covers both core contributor and external module developer workflows. Applies coding rules (PSR-12, SQL norms, MVC pattern, Active Record).'
---

# Dolibarr Developer Skill

## When to Use
- Creating or extending a Dolibarr module (custom or external)
- Implementing hooks (`actions_mymodule.class.php`) or triggers
- Writing DAO/business object classes, SQL table definitions
- Adding menus, tabs, permissions, boxes, exports, CSS/JS
- Building cron/command-line scripts
- Writing PDF/ODT document templates
- Following Dolibarr coding rules (PHP, SQL, HTML norms)

## Key Concepts

### Architecture
- **MVC pattern**: Controller (`/* Actions */`) + View (`/* View */`) in same PHP file
- **Active Record ORM**: one class per table with CRUD methods
- **Global objects**: `$db`, `$user`, `$conf`, `$langs`, `$mysoc`, `$hookmanager`, `$extrafields`
- **Module location**: external modules live in `htdocs/custom/mymodule/`
- **Table prefix**: all tables prefixed `llx_`

### Module Entry Points
| Extension point | When to use |
|---|---|
| Hooks | Inject/replace code in existing pages without modifying core |
| Triggers | React to business events (invoice created, order validated…) |
| Tabs | Add tabs on thirdparty, order, product, invoice… sheets |
| Menus | Add top-level or left-menu entries |
| Boxes | Add widgets on home page |
| Extrafields | Add fields to existing objects (no module needed for simple cases) |
| PDF templates | Customize generated PDF documents |

---

## Step-by-Step: Build a Module

### 1. Generate Skeleton
Use the built-in **Module Builder** (enable it in Setup → Modules, then click the bug icon top-right). It generates `modMyModule.class.php`, SQL files, DAO class, and page skeletons.

GitHub template: https://github.com/Dolibarr/dolibarr/tree/develop/htdocs/modulebuilder/template

### 2. Module Descriptor (required)
File: `htdocs/custom/mymodule/core/modules/modMyModule.class.php`
- Class name starts with `mod`, file matches class name
- Unique `$this->numero` (check https://wiki.dolibarr.org/index.php/List_of_modules_id)
- Declare hooks contexts, menus, permissions, tabs, boxes, CSS/JS in this file
- Enable/disable via Setup → Modules

See → [Module Structure reference](./references/module-structure.md)

### 3. SQL Tables (optional)
- Files in `mymodule/sql/llx_mytable.sql` + `llx_mytable.key.sql`
- Load via `$this->_load_tables('/mymodule/sql/')` in `init()`
- Primary key always `rowid INTEGER AUTO_INCREMENT PRIMARY KEY`
- Table prefix `llx_`, InnoDB engine, no DB triggers, no DELETE CASCADE

See → [Coding Rules reference](./references/coding-rules.md)

### 4. DAO Class (optional)
File: `mymodule/class/myobject.class.php`
- Copy from `htdocs/modulebuilder/templates/class/myobject.class.php`
- Methods: `create()`, `fetch()`, `update()`, `delete()`, `fetchAll()`
- DB access pattern: `$db->begin()` / `$db->query()` / `$db->commit()` or `$db->rollback()`

### 5. Hooks (optional)
- Declare context in `modMyModule.class.php`: `$this->module_parts = array('hooks' => array('thirdpartycard', 'orderlist'))`
- Create `mymodule/class/actions_mymodule.class.php` with hook methods
- **Disable + re-enable** module after changing contexts (stored in DB)
- Find contexts: search `initHooks(` in source. Find hook names: search `executeHooks(` in source.

See → [Hooks & Triggers reference](./references/hooks-triggers.md)

### 6. Menus (optional)
Declare in `$this->menu` array in module descriptor.
- `fk_menu=0` + `type='top'` for top menu
- `fk_menu='fk_mainmenu=xxx'` + `type='left'` for left sub-menu
- `perms` field controls visibility by permission
- `user=0` internal, `1` external, `2` both

### 7. Tabs (optional)
Declare in `$this->tabs` array: `'objecttype:+tabcode:Title:langfile@mymodule:condition:/mymodule/page.php?id=__ID__'`

Object types: `thirdparty`, `order`, `invoice`, `product`, `contact`, `contract`, `propal`, `member`, `user`…

### 8. Permissions (optional)
Declare in `$this->rights` array. Test with `$user->rights->mymodule->action->subaction`.

### 9. PHP Pages (optional)
- Bootstrap: include `main.inc.php` (try multiple relative paths, see template)
- Use `dol_include_once('/mymodule/class/myclass.class.php', 'MyClass')` for module classes
- Use `require_once DOL_DOCUMENT_ROOT.'/core/...'` for Dolibarr core classes
- CSS classes: `liste_titre`, `pair`/`impair`, `flat`, `button`
- JS: pass `$morejs` array to `llxHeader()`

### 10. Setup Page (optional)
- Create `mymodule/admin/setup.php`
- Set `$this->config_page_url = array("setup.php@mymodule")` in descriptor

---

## Packaging & Distribution
- Package via `build/makepack-dolibarrmodule.pl`
- Deploy: unzip in Dolibarr root
- Publish on https://www.dolistore.com

---

## References
- [Module structure, descriptor, file tree](./references/module-structure.md)
- [Hooks system & Triggers](./references/hooks-triggers.md)
- [Coding rules: PHP, SQL, HTML](./references/coding-rules.md)
- [Technical components: menus, tabs, permissions, DB, dates](./references/technical-components.md)

## Official Docs
- Developer docs: https://wiki.dolibarr.org/index.php/Developer_documentation
- Module development: https://wiki.dolibarr.org/index.php/Module_development
- Hooks system: https://wiki.dolibarr.org/index.php/Hooks_system
- Coding rules: https://wiki.dolibarr.org/index.php/Language_and_development_rules
- Doxygen (class/file tree): https://doxygen.dolibarr.org/
