# Dolibarr Hooks & Triggers Reference

Sources:
- https://wiki.dolibarr.org/index.php/Hooks_system
- https://wiki.dolibarr.org/index.php/Interfaces_Dolibarr_toward_foreign_systems

---

## Hooks vs Triggers

| | Hooks | Triggers |
|---|---|---|
| **Purpose** | Inject/replace code at any point in a page | React to business events (create/validate/delete) |
| **When called** | During page rendering or action handling | On specific Dolibarr events (BILL_CREATE, ORDER_VALIDATE…) |
| **Location** | Search `executeHooks(` in source | Search `run_triggers(` in source |
| **File** | `class/actions_mymodule.class.php` | `core/triggers/interface_99_modMyModule_*.class.php` |

---

## Hooks System

### Step 1 — Declare contexts in module descriptor

```php
$this->module_parts = array(
    'hooks' => array('thirdpartycard', 'orderlist', 'globalcard', 'all')
);
```

> **Important**: After adding/removing/renaming a context, you MUST disable + re-enable the module (contexts are stored in DB on activation).

Find all available contexts: search `initHooks(` in source files.
Common contexts:

| Context | Location |
|---|---|
| `thirdpartycard` | Third party view/edit card |
| `ordercard` | Customer order card |
| `invoicecard` | Customer invoice card |
| `productcard` | Product card |
| `contactcard` | Contact card |
| `contractcard` | Contract card |
| `propalcard` | Proposal/quote card |
| `membercard` | Foundation member card |
| `orderlist` | Customer order list |
| `invoicelist` | Invoice list |
| `globalcard` | All cards (global) |
| `all` | Every context |
| `main` | Every web page |
| `cli` | Every CLI script |

### Step 2 — Create the hook handler class

File: `htdocs/custom/mymodule/class/actions_mymodule.class.php`

```php
<?php
class ActionsMyModule
{
    public $errors = array();
    public $results = array();
    public $resprints = '';

    /**
     * Hook: doActions
     * Called during action processing (POST handling)
     * Return 0 = continue, 1 = replace standard code, <0 = error
     */
    public function doActions($parameters, &$object, &$action, $hookmanager)
    {
        if (in_array('thirdpartycard', explode(':', $parameters['context']))) {
            // your code here
        }
        return 0;
    }

    /**
     * Hook: formObjectOptions
     * Called to add HTML in the view form
     */
    public function formObjectOptions($parameters, &$object, &$action, $hookmanager)
    {
        if (in_array('productcard', explode(':', $parameters['context']))) {
            $this->resprints = '<tr><td>My extra field</td><td>value</td></tr>';
        }
        return 0;
    }

    /**
     * Hook: printFieldListSelect
     * Add fields to SQL SELECT in list queries
     */
    public function printFieldListSelect($parameters, &$object, &$action, &$hookmanager)
    {
        if (in_array('orderlist', explode(':', $parameters['context']))) {
            $this->resprints = ", myfield AS myalias";
        }
        return 0;
    }

    /**
     * Hook: printFieldListWhere
     * Add conditions to SQL WHERE in list queries
     */
    public function printFieldListWhere($parameters, &$object, &$action, $hookmanager)
    {
        return 0;
    }
}
```

### Return Codes

| Return | Meaning |
|---|---|
| `0` | Success; standard code following `if (empty($reshook))` WILL execute |
| `1` | Success; standard code following `if (empty($reshook))` will NOT execute (replaced) |
| `< 0` | Error; set `$this->errors[]` |

### Properties Set by Hook Handler

| Property | Effect |
|---|---|
| `$this->resprints` | String printed immediately after hook returns |
| `$this->results` | Array merged into `$hookmanager->resArray` |
| Modify `$object` | Changes propagate to caller |
| Modify `$action` | Changes propagate to caller |

### Finding Hook Names & Contexts
```bash
# Find all hook names
grep -r "executeHooks(" htdocs/ --include="*.php" | grep -o "'[^']*'" | sort -u

# Find all contexts
grep -r "initHooks(" htdocs/ --include="*.php"
```

---

## Common Hook Methods

| Hook method | Purpose |
|---|---|
| `doActions` | React to POST actions |
| `formObjectOptions` | Add HTML rows to view/edit form |
| `formCreateThirdpartyOptions` | Add fields in thirdparty create form |
| `printFieldListSelect` | Add SQL SELECT fields to list query |
| `printFieldListFrom` | Add SQL FROM/JOIN to list query |
| `printFieldListWhere` | Add SQL WHERE to list query |
| `printFieldListOrderby` | Add SQL ORDER BY to list query |
| `printFieldListTitle` | Add column header to list |
| `printFieldListValue` | Add column value to list |
| `addMoreBoxStatsCustomer` | Add stat boxes on thirdparty card |
| `formAddObjectLine` | Add rows to order/invoice line form |
| `afterPDFCreation` | Hook after PDF generation |
| `sendEmailsAfterSend` | Hook after email sent |

---

## Adding a Hook Point to Your Own Code

```php
// 1. Initialize hookmanager at top of page (after main.inc.php)
$hookmanager = new HookManager($db);
$hookmanager->initHooks(array('mymodulepage'));

// 2. Execute hooks at the point you want to be hookable
$parameters = array('mydata' => $value);
$reshook = $hookmanager->executeHooks('myHookName', $parameters, $object, $action);
if (empty($reshook)) {
    // standard code here — skipped if hook returns 1
    echo 'default output';
}
// Print what hooks added
echo $hookmanager->resprints;
```

---

## Triggers

### Trigger file naming convention
`htdocs/custom/mymodule/core/triggers/interface_99_modMyModule_MyTrigger.class.php`

Number `99` = priority (lower runs first).

### Trigger class structure

```php
<?php
require_once DOL_DOCUMENT_ROOT.'/core/triggers/dolibarrtriggers.class.php';

class InterfaceMyTrigger extends DolibarrTriggers
{
    public function __construct($db)
    {
        parent::__construct($db);
        $this->name        = 'MyTrigger';
        $this->description = 'Trigger for my module';
        $this->version     = '1.0';
        $this->picto       = 'technic';
    }

    public function runTrigger($action, $object, User $user, Translate $langs, Conf $conf)
    {
        if ($action == 'BILL_CREATE') {
            // react to invoice creation
            dol_syslog("Trigger BILL_CREATE fired", LOG_DEBUG);
        }
        if ($action == 'ORDER_VALIDATE') {
            // react to order validation
        }
        return 0; // 0=ok, <0=error
    }
}
```

### Common Trigger Events
`BILL_CREATE`, `BILL_MODIFY`, `BILL_VALIDATE`, `BILL_DELETE`, `BILL_SENTBYMAIL`
`ORDER_CREATE`, `ORDER_VALIDATE`, `ORDER_DELETE`, `ORDER_SENTBYMAIL`
`PROPAL_CREATE`, `PROPAL_VALIDATE`, `PROPAL_SENTBYMAIL`
`COMPANY_CREATE`, `COMPANY_MODIFY`, `COMPANY_DELETE`
`USER_CREATE`, `USER_MODIFY`, `USER_DELETE`
`CONTRACT_CREATE`, `CONTRACT_ACTIVATE`, `CONTRACT_CLOSE`
`PRODUCT_CREATE`, `PRODUCT_MODIFY`, `PRODUCT_DELETE`

Find all: search `run_triggers(` in source.
