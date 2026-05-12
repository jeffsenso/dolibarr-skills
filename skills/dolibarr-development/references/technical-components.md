# Dolibarr Technical Components Reference

Source: https://wiki.dolibarr.org/index.php/Developer_documentation

---

## Page Bootstrap (main.inc.php)

Every PHP page must load `main.inc.php`. Use the multi-path pattern:

```php
<?php
$res = 0;
if (!$res && !empty($_SERVER["CONTEXT_DOCUMENT_ROOT"])) {
    $res = @include($_SERVER["CONTEXT_DOCUMENT_ROOT"]."/main.inc.php");
}
$tmp = empty($_SERVER['SCRIPT_FILENAME']) ? '' : $_SERVER['SCRIPT_FILENAME'];
$tmp2 = realpath(__FILE__);
$i = strlen($tmp) - 1; $j = strlen($tmp2) - 1;
while ($i > 0 && $j > 0 && $tmp[$i] == $tmp2[$j]) { $i--; $j--; }
if (!$res && $i > 0 && file_exists(substr($tmp, 0, ($i+1))."/main.inc.php")) {
    $res = @include(substr($tmp, 0, ($i+1))."/main.inc.php");
}
if (!$res && file_exists("../main.inc.php"))   $res = @include("../main.inc.php");
if (!$res && file_exists("../../main.inc.php")) $res = @include("../../main.inc.php");
if (!$res) die("Include of main fails");
```

After include, available: `$db`, `$user`, `$conf`, `$langs`, `$mysoc`

---

## DAO / Business Object Class Pattern

```php
<?php
require_once DOL_DOCUMENT_ROOT.'/core/class/commonobject.class.php';

class MyObject extends CommonObject
{
    public $element    = 'myobject';
    public $table_element = 'mymodule_object';  // without llx_ prefix
    public $picto      = 'mymodule@mymodule';

    // Table fields (mirrors DB columns)
    public $ref;
    public $status;
    public $date_creation;
    public $fk_soc;         // foreign key to llx_societe
    // …

    public function __construct($db)
    {
        parent::__construct($db);
    }

    public function create(User $user, $notrigger = 0)
    {
        $error = 0;
        $this->db->begin();
        $sql = "INSERT INTO ".MAIN_DB_PREFIX."mymodule_object (ref, entity, fk_user_creat, date_creation)";
        $sql .= " VALUES ('".$this->db->escape($this->ref)."', ".((int) $this->entity).", ".((int) $user->id).", '".$this->db->idate(dol_now())."')";
        $res = $this->db->query($sql);
        if ($res) {
            $this->id = $this->db->last_insert_id(MAIN_DB_PREFIX."mymodule_object");
            if (!$error && !$notrigger) {
                $result = $this->call_trigger('MYOBJECT_CREATE', $user);
                if ($result < 0) { $error++; }
            }
        } else {
            $error++;
            $this->error = $this->db->lasterror();
        }
        if (!$error) { $this->db->commit(); return $this->id; }
        $this->db->rollback();
        return -1;
    }

    public function fetch($id, $ref = null)
    {
        $sql = "SELECT rowid, ref, status, date_creation, fk_soc";
        $sql .= " FROM ".MAIN_DB_PREFIX."mymodule_object";
        $sql .= " WHERE ";
        if ($id)  $sql .= "rowid = ".((int) $id);
        else       $sql .= "ref = '".$this->db->escape($ref)."'";
        $sql .= " AND entity = ".((int) $this->db->getEntity('myobject'));
        $resql = $this->db->query($sql);
        if ($resql) {
            $obj = $this->db->fetch_object($resql);
            if ($obj) {
                $this->id     = $obj->rowid;
                $this->ref    = $obj->ref;
                $this->status = $obj->status;
                $this->date_creation = $this->db->jdate($obj->date_creation);
                $this->fk_soc = $obj->fk_soc;
                return 1;
            }
            return 0;
        }
        $this->error = $this->db->lasterror();
        return -1;
    }

    public function update(User $user, $notrigger = 0) { /* … */ return 1; }
    public function delete(User $user, $notrigger = 0) { /* … */ return 1; }
}
```

---

## Tabs System

### Show tabs on your own page
```php
// 1. Include object class and lib
require_once DOL_DOCUMENT_ROOT.'/societe/class/societe.class.php';
require_once DOL_DOCUMENT_ROOT.'/core/lib/company.lib.php';

// 2. Load object
$id = GETPOST('id', 'int');
$thirdparty = new Societe($db);
$result = $thirdparty->fetch($id);

// 3. Get tab list
$head = societe_prepare_head($thirdparty);

// 4. Render tabs
dol_fiche_head($head, 'mytabcode', $langs->trans('ThirdParty'), -1, 'company');
// ... your content ...
dol_fiche_end();
```

Object-specific lib/function pairs:
| Object | Class file | Lib file | prepare_head function |
|---|---|---|---|
| Thirdparty | `societe/class/societe.class.php` | `core/lib/company.lib.php` | `societe_prepare_head()` |
| Product | `product/class/product.class.php` | `core/lib/product.lib.php` | `product_prepare_head()` |
| Invoice | `compta/facture/class/facture.class.php` | `core/lib/invoice.lib.php` | `facture_prepare_head()` |
| Order | `commande/class/commande.class.php` | `core/lib/order.lib.php` | `commande_prepare_head()` |
| Contact | `contact/class/contact.class.php` | `core/lib/contact.lib.php` | `contact_prepare_head()` |
| Contract | `contrat/class/contrat.class.php` | `core/lib/contract.lib.php` | `contract_prepare_head()` |
| Member | `adherents/class/adherent.class.php` | `core/lib/member.lib.php` | `member_prepare_head()` |

---

## Translation System

### Lang files
Location: `mymodule/langs/en_US/mymodule.lang`

```ini
MyKey=My translated string
MyKeyWithParam=Hello %s, you have %d messages
```

### Usage in PHP
```php
$langs->load('mymodule@mymodule');
echo $langs->trans('MyKey');
echo $langs->trans('MyKeyWithParam', $username, $count);
```

### Load in module descriptor
Auto-loaded from `$this->langfiles = array('mymodule@mymodule')` in descriptor.

---

## Permission System

### Check permissions in pages
```php
// Check user is logged in (done by main.inc.php)
// Check specific permission
if (!$user->rights->mymodule->read) {
    accessforbidden();
}
// For write
if (!$user->rights->mymodule->write) {
    accessforbidden();
}
```

### Admin-level checks
```php
if (!$user->admin) {
    accessforbidden();
}
```

---

## Configuration / Constants

### Read a constant
```php
// Constant stored in llx_const
$value = getDolGlobalString('MYMODULE_MYKEY');       // returns string, '' if not set
$value = getDolGlobalInt('MYMODULE_MYKEY');          // returns int, 0 if not set
$enabled = isModEnabled('mymodule');                  // check module enabled
```

### Save a constant (in setup page)
```php
dolibarr_set_const($db, 'MYMODULE_MYKEY', $value, 'chaine', 0, '', $conf->entity);
dolibarr_del_const($db, 'MYMODULE_MYKEY', $conf->entity);
```

---

## Forms & UI Helpers

### Form class
```php
$form = new Form($db);

// Select list
$form->select_thirdparty_list($selected_id, 'fk_soc', '', 1);

// Date picker
$form->select_date($timestamp, 'mydate', 0, 0, 0, 'myform');
// Then read back:
$mydate = dol_mktime(12, 0, 0,
    GETPOST('mydatemonth', 'int'),
    GETPOST('mydateday', 'int'),
    GETPOST('mydateyear', 'int')
);

// Status badge
$badge = $object->getLibStatut(5); // 5=full label with badge HTML
```

### Page wrapper
```php
$morejs  = array('/mymodule/js/mymodule.js');
$morecss = array('/mymodule/css/mymodule.css.php');
llxHeader('', $langs->trans('PageTitle'), '', '', '', '', $morejs, $morecss);
// ... content ...
llxFooter();
```

---

## Extrafields

### Add an extrafield via code (usually done via UI or install)
Managed in Setup → Display → Extrafields.

### Read/write extrafield values
```php
// After fetch(), extrafields are in $object->array_options
$val = $object->array_options['options_myfieldcode'];

// Fetch extrafields for an object
$extrafields = new ExtraFields($db);
$extrafields->fetch_name_optionals_label($object->table_element);
$object->fetch_optionals();

// Save extrafields
$object->insertExtraFields();
```

---

## URL & Path Helpers

```php
// Absolute URL from relative path
$url  = dol_buildpath('/mymodule/mypage.php', 1);   // 1=absolute URL
$path = dol_buildpath('/mymodule/mypage.php', 0);   // 0=filesystem path

// Image tag
echo img_picto('Alt text', 'myicon@mymodule', 'class="pictofixedwidth"');

// File path constants
DOL_DOCUMENT_ROOT   // htdocs/ filesystem path
DOL_URL_ROOT        // URL base (e.g. /dolibarr/htdocs)
DOL_DATA_ROOT       // documents/ directory path
```

---

## Error Reporting

```php
// In a class method
$this->error  = 'Error message';          // single error
$this->errors = array('err1', 'err2');    // multiple errors

// Display errors in page
if (!empty($object->errors)) {
    setEventMessages(null, $object->errors, 'errors');
}
setEventMessages($langs->trans('RecordSaved'), null, 'mesgs');
setEventMessages($langs->trans('Warning'), null, 'warnings');
```

---

## Cron / CLI Scripts

CLI scripts go in `mymodule/scripts/mymodule_cron.php`.

```php
#!/usr/bin/env php
<?php
// Load Dolibarr environment
if (!defined('NOREQUIREUSER'))  define('NOREQUIREUSER', '1');
if (!defined('NOREQUIREMENU'))  define('NOREQUIREMENU', '1');
if (!defined('NOREQUIREHTML'))  define('NOREQUIREHTML', '1');
// … find and include main.inc.php (same multi-path pattern as pages)

/**
 * Function called by cron job
 * @return int 0=OK, <0=error
 */
function mymodule_cron_function()
{
    global $db, $conf, $langs, $user;
    // your logic
    return 0;
}
```

Register in module descriptor:
```php
$this->cronjobs = array(
    0 => array(
        'label'     => 'My cron task',
        'jobtype'   => 'function',
        'class'     => '/mymodule/class/myobject.class.php',
        'objectname' => 'MyObject',
        'method'    => 'myCronMethod',
        'parameters' => '',
        'comment'   => 'Description',
        'frequency' => 1,
        'unitfrequency' => 3600,  // seconds
        'status'    => 0,
        'test'      => '$conf->mymodule->enabled',
    ),
);
```

---

## REST API Integration

Dolibarr exposes REST API. Add your own API endpoint:

File: `mymodule/class/api_mymodule.class.php`
Extends: `DolibarrApi`

Enable via Setup → API.
Docs: https://wiki.dolibarr.org/index.php/Module_Web_Services_API_REST_(developer)
