# Dolibarr Coding Rules Reference

Source: https://wiki.dolibarr.org/index.php/Language_and_development_rules

---

## PHP Rules

### Compatibility
- PHP 7.1.0+ (no required extra modules except DB driver)
- MySQL 5.7+ / MariaDB. PostgreSQL supported (SQL converted on the fly by driver)
- Must work on all OS (Windows, Linux, macOS)

### File Conventions
- All PHP files end with `.php`
- Files saved in Unix format (LF, not CR/LF)
- Always use `<?php` — never short tags `<?` or `<?=`
- Copyright header required at top of every file

```php
<?php
/* Copyright (C) 2024 Your Name <email@example.com>
 *
 * This program is free software; you can redistribute it and/or modify
 * it under the terms of the GNU General Public License as published by
 * the Free Software Foundation; either version 3 of the License.
 */
```

### Coding Style
- **PSR-12** "MUST" rules apply (https://www.php-fig.org/psr/psr-12/)
- Exceptions: tabs allowed (don't replace with spaces), long lines acceptable for data declarations, hard limit 1000 chars/line
- Variables outside strings: `"text ".$variable." !\n"` not `"text $variable !\n"`
- Comments: C-style (`//` single line, `/* */` blocks)
- Functions return `>= 0` on success, `< 0` on error
- Use `include_once` for files with class/function definitions (`*.class.php`, `*.lib.php`)
- Use `include` for template-style files (`*.inc.php`, `*.tpl.php`)
- No dead code in core; no `SELECT *`

### User Input — always use GETPOST
```php
// NEVER use $_GET/$_POST directly
$id      = GETPOST('id', 'int');
$ref     = GETPOST('ref', 'alpha');
$mytext  = GETPOST('mytext', 'alphanohtml');
// Sanitized $_SERVER["PHP_SELF"] is handled by main.inc.php
```

### Global Variables (always available after main.inc.php)
```php
$db          // Database connection handler
$user        // Current user object
$conf        // Configuration object
$langs       // Language/translation object
$mysoc       // Current company object
$hookmanager // Hook factory
$extrafields // Extrafields factory
```

### Including Files
```php
// Core Dolibarr class
require_once DOL_DOCUMENT_ROOT.'/core/class/html.form.class.php';
// Module class (use dol_include_once for module files)
dol_include_once('/mymodule/class/myobject.class.php', 'MyObject');
```

### Logging
```php
dol_syslog("MyModule: action done", LOG_INFO);
dol_syslog("MyModule: debug detail", LOG_DEBUG);
dol_syslog("MyModule: warning", LOG_WARNING);
dol_syslog("MyModule: error: ".$this->error, LOG_ERR);
```

### Dates — use Dolibarr functions only
```php
// Current timestamp (GMT)
$now = dol_now();

// Build a date from parts
$date = dol_mktime($hour, $min, $sec, $month, $day, $year);

// Format for display
$formatted = dol_print_date($timestamp, 'day');        // date only
$formatted = dol_print_date($timestamp, 'dayhour');    // date + time

// String to timestamp
$ts = dol_stringtotime('2024-01-15');

// Date arithmetic
$future = dol_time_plus_duree($now, 1, 'm'); // +1 month

// For SQL — convert timestamp to DB string and back
$sqlval = $db->idate($timestamp);   // timestamp → DB string
$tsback = $db->jdate($dbstring);    // DB string → timestamp
```

> Dates in DB are stored in PHP server timezone. Fields `tms` (auto-updated) are GMT.

### Amounts & Float Numbers
```php
// ALWAYS clean float results with price2num
$total = price2num($unitprice * $qty, 'MT');  // MU=unit price, MT=total, MS=other
// For non-amounts use round()
$qty = round($calculated_qty, 2);
```

### Working Directories
```php
$dir = DOL_DATA_ROOT.'/mymodule';
dol_mkdir($dir);
$tmpdir = DOL_DATA_ROOT.'/mymodule/temp';
```

### Version Comparison
```php
if ((float) DOL_VERSION >= 17.0) {
    // code for Dolibarr 17+
}
// Or with full version:
$version = preg_split('/[\.-]/', DOL_VERSION);
if (versioncompare($version, array(17, 0, 0)) >= 0) { ... }
```

---

## SQL Rules

### Table Naming & Structure
- Prefix: `llx_` (e.g. `llx_mymodule_object`)
- Engine: InnoDB only
- Always define `rowid INTEGER NOT NULL AUTO_INCREMENT PRIMARY KEY`
- Standard fields to include:

```sql
CREATE TABLE llx_mymodule_object (
    rowid        integer NOT NULL AUTO_INCREMENT PRIMARY KEY,
    ref          varchar(30)  NOT NULL,
    entity       integer DEFAULT 1 NOT NULL,  -- multicompany
    ref_ext      varchar(255),                -- external system ref
    -- your fields here
    date_creation datetime NOT NULL,
    tms          timestamp,                   -- auto-updated by DB
    fk_user_creat integer NOT NULL,
    fk_user_modif integer,
    import_key   varchar(14),
    status       smallint DEFAULT 0 NOT NULL,
    note_private text,
    note_public  text
) ENGINE=InnoDB;
```

### Field Types
| Use case | Type |
|---|---|
| Primary / foreign key | `integer` (or `bigint` for large tables) |
| Boolean / small number | `smallint` |
| Amount | `double(24,8)` |
| VAT rate | `double(6,3)` |
| Quantity | `real` |
| String | `varchar(N)` |
| Date+time (auto) | `timestamp` |
| Date+time | `datetime` |
| Date only | `date` |
| Large text | `text` or `mediumtext` |

No `enum`, no `char(1)` (use `varchar`).

### Keys
- Primary: `rowid`
- Unique keys: `uk_tablename_field`
- Foreign keys: `fk_tablename_fieldname` — **soft FK only** (managed by PHP, no DB constraints to external tables)
- Performance indexes: `idx_tablename_fieldname`
- Key files: `llx_mytable.key.sql`

```sql
ALTER TABLE llx_mymodule_object ADD UNIQUE uk_mymodule_object_ref (ref, entity);
ALTER TABLE llx_mymodule_object ADD INDEX idx_mymodule_object_status (status);
```

### SQL Coding
```php
// Transactions
$db->begin();
$result = $db->query("INSERT INTO llx_mymodule_object (...) VALUES (...)");
if ($result) {
    $db->commit();
} else {
    $db->rollback();
    $this->error = $db->lasterror();
    return -1;
}

// SELECT — no SELECT *, no SQL date functions
$sql  = "SELECT rowid, ref, status";
$sql .= " FROM ".MAIN_DB_PREFIX."mymodule_object";
$sql .= " WHERE entity = ".((int) $conf->entity);
$sql .= " AND status = ".((int) $status);
$sql .= " ORDER BY ref ASC";

$resql = $db->query($sql);
if ($resql) {
    $num = $db->num_rows($resql);
    while ($obj = $db->fetch_object($resql)) {
        echo $obj->ref;
    }
    $db->free($resql);
}

// Dates in SQL — use PHP value, not SQL NOW()
$sql .= " AND date_creation >= '".$db->idate(dol_now() - 86400 * 7)."'";

// SQL IF — use $db->ifsql() for portability
$sql .= ", ".$db->ifsql("status = 1", "'active'", "'inactive'")." AS status_label";
```

### Forbidden
- `SELECT *`
- `NOW()`, `SYSDATE()`, `DATEDIFF()`, `DATE()` in SQL (use PHP `dol_now()` + `$db->idate()`)
- `GROUP_CONCAT`
- `WITH ROLLUP`
- `DELETE CASCADE` / `ON UPDATE CASCADE` (between core tables)
- Database triggers / stored procedures
- Quoting numeric values in INSERT/UPDATE

---

## HTML Rules

- HTML compliant (not XHTML); attributes lowercase and double-quoted
- Use `dol_buildpath()` for absolute URLs
- Use `img_picto()` for images
- No forced column widths unless content length is known
- JavaScript: wrap in `if ($conf->use_javascript_ajax) { ... }`
- No popup windows (except tooltips)
- No external template frameworks (Smarty, Twig…) — use `.tpl.php` files

### Standard CSS Classes
| Class | Use |
|---|---|
| `liste_titre` | Header row of a table (`<tr>` and `<td>`) |
| `pair` / `impair` | Alternating rows |
| `flat` | Input fields (input, select, textarea) |
| `button` | Submit buttons |

### MVC Structure in PHP Pages
```php
/* Actions (Controller) */
if ($action == 'save') {
    // handle POST
}

/* View */
llxHeader('', $langs->trans('MyPage'), '', '', '', '', $morejs, $morecss);
// output HTML
llxFooter();
```

---

## Design Patterns Used in Dolibarr
- **Table Module** (Martin Fowler): one class per table
- **Active Record**: CRUD methods in the class + business logic
- **MVC**: Controller (`/* Actions */`) + View (`/* View */`) in same PHP file, separated by comment tags
- No Composer for deployment; external libs embedded manually in source
