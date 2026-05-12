# Dolibarr Module Structure Reference

Source: https://wiki.dolibarr.org/index.php/Module_development

---

## File Tree of an External Module

```
htdocs/custom/mymodule/
├── core/
│   ├── modules/
│   │   └── modMyModule.class.php        ← Module descriptor (REQUIRED)
│   ├── triggers/
│   │   └── interface_99_modMyModule_*.class.php
│   ├── boxes/
│   │   └── mybox.php
│   └── tpl/                             ← Override .tpl files (requires 'tpl'=>1)
├── admin/
│   └── setup.php                        ← Setup page
├── class/
│   ├── myobject.class.php               ← DAO / business object
│   └── actions_mymodule.class.php       ← Hook handler
├── css/
│   └── mymodule.css.php
├── js/
│   └── mymodule.js
├── langs/
│   └── en_US/
│       └── mymodule.lang
├── sql/
│   ├── llx_mytable.sql
│   ├── llx_mytable.key.sql
│   └── data.sql
├── img/
├── lib/
├── scripts/                             ← CLI scripts (must start with #!/usr/bin/env php)
├── docs/
└── mypage.php                           ← PHP pages
```

---

## Module Descriptor: modMyModule.class.php

Minimum required properties:

```php
class modMyModule extends DolibarrModules
{
    public function __construct($db)
    {
        $this->db = $db;
        $this->numero       = 500000;     // Unique ID — check wiki list
        $this->rights_class = 'mymodule';
        $this->family       = 'other';
        $this->name         = 'MyModule';
        $this->description  = 'My module description';
        $this->version      = '1.0.0';    // or 'experimental' / 'development'
        $this->const_name   = 'MAIN_MODULE_MYMODULE';
        $this->picto        = 'mymodule@mymodule';

        // Hooks contexts this module hooks into
        $this->module_parts = array(
            'hooks'  => array('thirdpartycard', 'orderlist', 'globalcard'),
            'css'    => array('/mymodule/css/mymodule.css.php'),
            'js'     => array('/mymodule/js/mymodule.js'),
            'tpl'    => 1,                // allow .tpl overrides
        );

        // SQL tables to create on activation
        // (called in init())
        // $this->_load_tables('/mymodule/sql/');

        // Menus
        $this->menu = array();
        // see Menus section below

        // Permissions
        $this->rights = array();
        // see Permissions section below

        // Tabs on existing objects
        $this->tabs = array();
        // see Tabs section below

        // Boxes
        $this->boxes = array();
        // see Boxes section below

        // Exports
        $this->export_code       = array();
        $this->export_label      = array();
        $this->export_icon       = array();
        $this->export_fields_array = array();

        // Config page
        $this->config_page_url = array("setup.php@mymodule");
    }

    public function init($options = '')
    {
        $this->_load_tables('/mymodule/sql/');
        return $this->_init(array(), $options);
    }

    public function remove($options = '')
    {
        $sql = array();
        return $this->_remove($sql, $options);
    }
}
```

---

## Menus Declaration

```php
$r = 0;
// Top menu
$this->menu[$r] = array(
    'fk_menu'  => 0,
    'type'     => 'top',
    'titre'    => 'MyModule',
    'mainmenu' => 'mymodule',
    'leftmenu' => 'mymodule',
    'url'      => '/mymodule/index.php',
    'langs'    => 'mymodule@mymodule',
    'position' => 100,
    'enabled'  => '$conf->mymodule->enabled',
    'perms'    => '1',
    'target'   => '',
    'user'     => 2,    // 0=internal, 1=external, 2=both
);
$r++;

// Left sub-menu
$this->menu[$r] = array(
    'fk_menu'  => 'fk_mainmenu=mymodule',
    'type'     => 'left',
    'titre'    => 'MyList',
    'mainmenu' => 'mymodule',
    'leftmenu' => 'mymodulelist',
    'url'      => '/mymodule/list.php',
    'langs'    => 'mymodule@mymodule',
    'position' => 100,
    'enabled'  => '$conf->mymodule->enabled',
    'perms'    => '$user->rights->mymodule->read',
    'target'   => '',
    'user'     => 2,
);
$r++;
```

---

## Tabs Declaration

```php
$this->tabs = array(
    // Add a tab on thirdparty sheet
    'thirdparty:+mytab:MyTabTitle:mymodule@mymodule:$user->rights->mymodule->read:/mymodule/tab.php?id=__ID__',
    // Remove an existing tab
    // 'thirdparty:-tabname',
);
```

Available object types: `thirdparty`, `order`, `invoice`, `supplier_order`, `supplier_invoice`, `product`, `stock`, `propal`, `member`, `contract`, `user`, `group`, `contact`, `payment`, `payment_supplier`, `categories_x`

---

## Permissions Declaration

```php
$r = 0;
$this->rights[$r][0] = 50001;           // Unique permission ID
$this->rights[$r][1] = 'Read objects';  // Default label (translation key: Permission50001)
$this->rights[$r][2] = 'r';             // r=read, w=write, d=delete
$this->rights[$r][3] = 1;               // 1=granted to new users by default
$this->rights[$r][4] = 'read';          // action
// $this->rights[$r][5] = 'subaction'; // optional subaction
$r++;
```

Test in code: `if ($user->rights->mymodule->read) { ... }`

---

## Boxes Declaration

```php
$this->boxes[0]['file'] = 'mybox0.php@mymodule';
$this->boxes[0]['note'] = 'My box description';
```

Create file: `mymodule/core/boxes/mybox0.php` (copy from `htdocs/core/boxes/` as example)

---

## Module ID List
Reserve your module ID: https://wiki.dolibarr.org/index.php/List_of_modules_id

External modules: use IDs >= 500000 (or a block reserved for you).

---

## Module Builder
Enable: Setup → Modules → search "Module Builder" → activate.
Then click the bug icon (top right). Generates descriptor, SQL, DAO class, and pages automatically.
