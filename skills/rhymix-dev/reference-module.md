# Rhymix Module Development Reference

## Directory Structure (Modern Namespace-based)

IMPORTANT: `info.xml` and `module.xml` MUST be inside the `conf/` subdirectory, NEVER in the module root.

```
modules/{module_name}/
  composer.json                        # Autoloader config - REQUIRED
  conf/                                # Config directory - REQUIRED
    info.xml                           # Module metadata - REQUIRED (MUST be in conf/)
    module.xml                         # Action/grant/trigger definitions - REQUIRED (MUST be in conf/)
  controllers/
    Base.php                           # Base class (extends \ModuleObject) - REQUIRED
    Install.php                        # Install/update callbacks - REQUIRED
    Index.php                          # Default frontend controller
    Admin.php                          # Admin controller
    EventHandlers.php                  # Trigger handlers
    {Feature}.php                      # Feature-grouped controllers (e.g., Read.php, Write.php)
  models/
    Config.php                         # Module configuration model
    {ModelName}.php                    # Other models
  lang/
    ko.php                             # Korean language
    en.php                             # English language
  schemas/                             # DB table definitions (XML)
  queries/                             # DB query definitions (XML)
  ruleset/                             # Form validation rules (XML)
  skins/                               # Frontend skins
    default/
      skin.xml                         # Skin metadata and extra_vars
      index.html                       # Templates (.html or .blade.php)
      css/skin.scss
      js/skin.js
  m.skins/                             # Mobile skins (same structure as skins/)
    default/
      skin.xml
      index.html
      css/skin.scss
      js/skin.js
  views/                               # Admin view templates
    admin/
      config.blade.php                 # Admin templates (use v2 / .blade.php)
      config.scss
      config.js
  .editorconfig                        # Editor config
  .gitignore
  LICENSE
  README.md
```

## composer.json

Every modern Rhymix module requires this for PSR-4 autoloading:

```json
{
  "config": {
    "optimize-autoloader": true,
    "prepend-autoloader": false
  },
  "require": {
    "rhymix/composer-stub": "dev-master"
  }
}
```

## conf/info.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<module version="0.2">
    <title xml:lang="ko">Module Name</title>
    <description xml:lang="ko">Module description</description>
    <version>1.0.0</version>
    <date>2024-01-01</date>
    <category>service</category>
    <author email_address="dev@example.com" link="https://example.com">
        <name xml:lang="ko">Author Name</name>
    </author>
</module>
```

## conf/module.xml (Modern Format)

The modern format uses `class` attribute pointing to namespaced controller classes instead of the old `type` attribute.

```xml
<?xml version="1.0" encoding="utf-8"?>
<module>
    <grants>
        <grant name="list" default="guest">
            <title xml:lang="ko">List</title>
            <title xml:lang="en">List</title>
        </grant>
        <grant name="write" default="member">
            <title xml:lang="ko">Write</title>
            <title xml:lang="en">Write</title>
        </grant>
    </grants>

    <actions>
        <!-- Admin actions -->
        <action name="dispMyModuleAdminConfig" class="Controllers\Admin"
                menu-name="mymodule" admin-index="true" />
        <action name="procMyModuleAdminInsertConfig" class="Controllers\Admin" />

        <!-- Frontend view actions (index="true" = default action) -->
        <action name="dispMyModuleIndex" class="Controllers\Index" index="true" />

        <!-- Read action with route -->
        <action name="dispMyModuleRead" class="Controllers\Read"
                route="read/$item_srl" />

        <!-- Write actions with multiple routes -->
        <action name="dispMyModuleWrite" class="Controllers\Write">
            <route route="write" />
            <route route="edit/$item_srl" />
        </action>

        <!-- Controller (POST) action with permission -->
        <action name="procMyModuleWrite" class="Controllers\Write"
                permission="write" />
    </actions>

    <!-- Event handlers (triggers) -->
    <eventHandlers>
        <eventHandler after="document.insertDocument"
                      class="Controllers\EventHandlers" method="afterInsertDocument" />
        <eventHandler after="document.updateDocument"
                      class="Controllers\EventHandlers" method="afterUpdateDocument" />
        <eventHandler after="document.deleteDocument"
                      class="Controllers\EventHandlers" method="afterDeleteDocument" />
    </eventHandlers>

    <!-- Admin menus -->
    <menus>
        <menu name="mymodule" type="all">
            <title xml:lang="ko">My Module</title>
            <title xml:lang="en">My Module</title>
        </menu>
    </menus>
</module>
```

### Key Differences from Legacy (XE-style) module.xml

| Aspect | Legacy (XE) | Modern (Rhymix) |
|--------|-------------|-----------------|
| Action routing | `type="view"`, `type="controller"` | `class="Controllers\ClassName"` |
| eventHandler class | `class="controller"` | `class="Controllers\EventHandlers"` |
| Menu attribute | `menu_name` | `menu-name` |
| Admin index | `admin_index="true"` | `admin-index="true"` |

### Action Attribute Reference

| Attribute | Description |
|-----------|-------------|
| `class` | Namespaced controller class (e.g., `Controllers\Admin`) |
| `permission` | `guest`, `member`, `manager`, `root`, or grant name |
| `index="true"` | Default frontend action |
| `admin-index="true"` | Default admin action |
| `menu-name` | Associates action with an admin menu |
| `route` | Short URL route pattern (inline attribute or child elements) |
| `error-handlers` | e.g., `"404"` |
| `global-route` | `"true"` for mid-less routing |

### Route Variable Types

| Type | Description |
|------|-------------|
| `int` | Non-negative integer |
| `float` | Non-negative decimal |
| `alpha` | Alphabetic characters only |
| `alnum` | Alphanumeric characters |
| `hex` | Hexadecimal (0-9, a-f) |
| `word` | Variable-name characters (alphanumeric + underscore) |
| `any` | All characters except slashes (default) |
| `delete` | Removes variable from URL |

Note: Variables ending in `_srl` default to `int` type.

Multiple routes per action use child `<route>` elements. Priority attribute resolves conflicts.

## Class Structure (Namespace-based)

Namespace pattern: `Rhymix\Modules\{ModuleName}\Controllers` and `Rhymix\Modules\{ModuleName}\Models`

### Base Class (controllers/Base.php)

```php
<?php

namespace Rhymix\Modules\MyModule\Controllers;

/**
 * MyModule base controller.
 */
class Base extends \ModuleObject
{

}
```

### Install Class (controllers/Install.php)

```php
<?php

namespace Rhymix\Modules\MyModule\Controllers;

/**
 * MyModule install/update handler.
 */
class Install extends Base
{
    /**
     * Module install callback.
     *
     * @return object
     */
    public function moduleInstall()
    {

    }

    /**
     * Check if the module needs to be updated.
     *
     * @return bool
     */
    public function checkUpdate()
    {

    }

    /**
     * Module update callback.
     *
     * @return object
     */
    public function moduleUpdate()
    {

    }

    /**
     * Cache recompile callback.
     *
     * @return void
     */
    public function recompileCache()
    {

    }
}
```

### Frontend Controller (controllers/Index.php)

```php
<?php

namespace Rhymix\Modules\MyModule\Controllers;

/**
 * MyModule default frontend controller.
 */
class Index extends Base
{
    /**
     * Initialize.
     */
    public function init()
    {
        $this->setTemplatePath($this->module_path . 'skins/' . ($this->module_info->skin ?: 'default'));
    }

    /**
     * Display main page.
     */
    public function dispMyModuleIndex()
    {
        $this->setTemplateFile('index');
    }
}
```

### Feature Controller (controllers/Write.php)

In the modern structure, related view (disp) and controller (proc) actions can be grouped in the same class file:

```php
<?php

namespace Rhymix\Modules\MyModule\Controllers;

/**
 * MyModule write controller.
 */
class Write extends Base
{
    /**
     * Initialize.
     */
    public function init()
    {
        $this->setTemplatePath($this->module_path . 'skins/' . ($this->module_info->skin ?: 'default'));
    }

    /**
     * Display write form.
     */
    public function dispMyModuleWrite()
    {
        $this->setTemplateFile('write');
    }

    /**
     * Process write action.
     */
    public function procMyModuleWrite()
    {
        if (!$this->grant->write)
        {
            throw new \Rhymix\Framework\Exceptions\NotPermitted;
        }

        $obj = \Context::getRequestVars();
        $obj->module_srl = $this->module_srl;
        $obj->item_srl = getNextSequence();

        $output = executeQuery('mymodule.insertItem', $obj);
        if (!$output->toBool())
        {
            return $output;
        }

        $this->setMessage('success_registed');
        $this->setRedirectUrl(getNotEncodedUrl('', 'mid', $this->mid));
    }
}
```

### Admin Controller (controllers/Admin.php)

```php
<?php

namespace Rhymix\Modules\MyModule\Controllers;

use Rhymix\Modules\MyModule\Models\Config as ConfigModel;
use Context;
use BaseObject;

/**
 * MyModule admin controller.
 */
class Admin extends Base
{
    /**
     * Initialize.
     */
    public function init()
    {
        $this->setTemplatePath($this->module_path . 'views/admin/');
    }

    /**
     * Display admin config.
     */
    public function dispMyModuleAdminConfig()
    {
        $config = ConfigModel::getConfig();
        Context::set('config', $config);
        $this->setTemplateFile('config');
    }

    /**
     * Save admin config.
     */
    public function procMyModuleAdminInsertConfig()
    {
        $config = ConfigModel::getConfig();
        $vars = Context::getRequestVars();

        if (in_array($vars->example_config, ['Y', 'N']))
        {
            $config->example_config = $vars->example_config;
        }
        else
        {
            return new BaseObject(-1, 'invalid_config_value');
        }

        $output = ConfigModel::setConfig($config);
        if (!$output->toBool())
        {
            return $output;
        }

        $this->setMessage('success_registed');
        $this->setRedirectUrl(Context::get('success_return_url'));
    }
}
```

### Event Handlers (controllers/EventHandlers.php)

```php
<?php

namespace Rhymix\Modules\MyModule\Controllers;

/**
 * MyModule event handlers (triggers).
 */
class EventHandlers extends Base
{
    /**
     * After document insert trigger.
     *
     * @param object $obj
     */
    public function afterInsertDocument($obj)
    {

    }

    /**
     * After document update trigger.
     *
     * @param object $obj
     */
    public function afterUpdateDocument($obj)
    {

    }

    /**
     * After document delete trigger.
     *
     * @param object $obj
     */
    public function afterDeleteDocument($obj)
    {

    }
}
```

### Config Model (models/Config.php)

```php
<?php

namespace Rhymix\Modules\MyModule\Models;

use ModuleController;
use ModuleModel;

/**
 * MyModule configuration model.
 */
class Config
{
    /**
     * Cache for module config.
     */
    protected static $_cache = null;

    /**
     * Get module config.
     *
     * @return object
     */
    public static function getConfig()
    {
        if (self::$_cache === null)
        {
            self::$_cache = ModuleModel::getModuleConfig('mymodule') ?: new \stdClass;
        }
        return self::$_cache;
    }

    /**
     * Save module config.
     *
     * @param object $config
     * @return object
     */
    public static function setConfig($config)
    {
        $oModuleController = ModuleController::getInstance();
        $result = $oModuleController->insertModuleConfig('mymodule', $config);
        if ($result->toBool())
        {
            self::$_cache = $config;
        }
        return $result;
    }
}
```

## Template File Conventions

| Location | Extension | Syntax |
|----------|-----------|--------|
| `views/admin/` | `.blade.php` | Template v2 (Blade-style) |
| `skins/*/` | `.html` or `.blade.php` | v1 or v2 (v2 preferred for new modules) |
| `m.skins/*/` | `.html` or `.blade.php` | v1 or v2 (v2 preferred for new modules) |

Admin views MUST use `.blade.php` (v2). For skins, `.html` with `<config autoescape="on" />` is still common, but `.blade.php` (v2) is preferred for new development.

## ModuleObject Properties and Methods

### Properties

| Property | Description |
|----------|-------------|
| `$this->module` | Module name string |
| `$this->module_info` | Current module instance configuration |
| `$this->module_config` | Module-wide configuration |
| `$this->module_path` | Filesystem path to module directory |
| `$this->module_srl` | Current module instance serial number |
| `$this->mid` | Current module instance identifier |
| `$this->act` | Current action name |
| `$this->template_path` | Template path |
| `$this->template_file` | Template file |
| `$this->user` | Logged-in member info |
| `$this->grant` | Permission grant object for current user |

### Methods

| Method | Description |
|--------|-------------|
| `setTemplatePath($path)` | Set output template path |
| `setTemplateFile($file)` | Set output template file |
| `setRedirectUrl($url)` | Redirect URL after controller action |
| `setLayoutPath($path)` | Set layout path |
| `setLayoutFile($file)` | Set layout file |
| `setMessage($msg)` | Set result message |
