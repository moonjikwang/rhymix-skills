# Rhymix Module Development Reference

## Directory Structure

```
modules/{module_name}/
  {module_name}.class.php              # Main class (extends ModuleObject) - REQUIRED
  {module_name}.controller.php         # Controller (proc* actions)
  {module_name}.model.php              # Model (get* actions)
  {module_name}.view.php               # View (disp* actions)
  {module_name}.admin.controller.php   # Admin controller
  {module_name}.admin.model.php        # Admin model
  {module_name}.admin.view.php         # Admin view
  {module_name}.mobile.php             # Mobile view (extends View class)
  {module_name}.api.php                # API class
  conf/
    info.xml                           # Module metadata - REQUIRED
    module.xml                         # Action/grant/trigger definitions - REQUIRED
  lang/
    ko.php                             # Korean
    en.php                             # English
  schemas/                             # DB table definitions (XML)
  queries/                             # DB query definitions (XML)
  ruleset/                             # Form validation rules (XML)
  skins/                               # Frontend skins
    {skin_name}/
      skin.xml                         # Skin metadata and extra_vars
      *.html                           # Templates
  m.skins/                             # Mobile skins
  tpl/                                 # Admin templates
```

## conf/info.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<module version="0.2">
    <title xml:lang="ko">모듈 이름</title>
    <title xml:lang="en">Module Name</title>
    <description xml:lang="ko">모듈 설명</description>
    <description xml:lang="en">Module description</description>
    <version>1.0.0</version>
    <date>2024-01-01</date>
    <category>service</category>
    <author email_address="dev@example.com" link="https://example.com/">
        <name xml:lang="ko">Author Name</name>
        <name xml:lang="en">Author Name</name>
    </author>
</module>
```

## conf/module.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<module>
    <!-- Permission grants -->
    <grants>
        <grant name="list" default="guest">
            <title xml:lang="ko">목록</title>
            <title xml:lang="en">List</title>
        </grant>
        <grant name="write" default="member">
            <title xml:lang="ko">작성</title>
            <title xml:lang="en">Write</title>
        </grant>
    </grants>

    <!-- Action definitions -->
    <actions>
        <!-- View actions: disp{ModuleName}{Action} -->
        <action name="dispMyModuleList" type="view" permission="list"
                standalone="false" index="true">
            <route route="page/$page:int" priority="10" />
        </action>

        <action name="dispMyModuleWrite" type="view" permission="write"
                standalone="false" meta-noindex="true">
            <route route="write" />
            <route route="$item_srl/edit" />
        </action>

        <!-- Controller actions: proc{ModuleName}{Action} -->
        <action name="procMyModuleInsert" type="controller"
                permission="write" standalone="false"
                ruleset="insertItem" />

        <!-- Admin actions -->
        <action name="dispMyModuleAdminList" type="view"
                admin_index="true" menu_name="mymodule" menu_index="true" />

        <action name="dispMyModuleAdminConfig" type="view"
                setup_index="true" menu_name="mymodule" />

        <!-- Mobile actions -->
        <action name="dispMyModuleList" type="mobile"
                permission="list" standalone="false" />

        <!-- 404 error handler -->
        <action name="dispMyModuleNotFound" type="view"
                standalone="false" error-handlers="404" />
    </actions>

    <!-- Event handlers (triggers) -->
    <eventHandlers>
        <eventHandler after="document.insertDocument"
                      class="controller" method="triggerAfterInsertDocument" />
        <eventHandler before="module.dispAdditionSetup"
                      class="model" method="triggerDispAdditionSetup" />
    </eventHandlers>

    <!-- Admin menus -->
    <menus>
        <menu name="mymodule" type="all">
            <title xml:lang="ko">내 모듈</title>
            <title xml:lang="en">My Module</title>
        </menu>
    </menus>
</module>
```

### Action Attribute Reference

| Attribute | Description |
|-----------|-------------|
| `type` | `view`, `controller`, `model`, `mobile`, `api` |
| `permission` | `guest`, `member`, `manager`, `root`, or grant name |
| `standalone` | `true`/`false` — whether action works without a module instance |
| `index="true"` | Default frontend action |
| `admin_index="true"` | Default admin action |
| `setup_index="true"` | Module setup entry point |
| `menu_name` | Associates action with an admin menu |
| `menu_index="true"` | Default action for that admin menu |
| `ruleset` | XML filename in `ruleset/` directory (without .xml) |
| `check_csrf` | `"false"` to disable CSRF check |
| `method` | Allowed HTTP methods (e.g., `"GET\|POST"`) |
| `meta-noindex` | `"true"` to add noindex meta tag |
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

### eventHandler Attributes

- `before` or `after`: trigger point as `module_name.trigger_name`
- `class`: handler class type (`controller`, `model`, `view`)
- `method`: method name to call

## Class Structure

### Main Class ({module_name}.class.php)

```php
<?php

class MyModule extends ModuleObject
{
    /**
     * Install the module.
     *
     * @return BaseObject|void
     */
    function moduleInstall()
    {
    }

    /**
     * Check if the module needs to be updated.
     *
     * @return bool
     */
    function checkUpdate()
    {
        return false;
    }

    /**
     * Update the module.
     *
     * @return BaseObject|void
     */
    function moduleUpdate()
    {
    }

    /**
     * Uninstall the module.
     *
     * @return BaseObject|void
     */
    function moduleUninstall()
    {
    }
}
```

### View Class ({module_name}.view.php)

```php
<?php

class MyModuleView extends MyModule
{
    /**
     * Initialize the view.
     *
     * @return void
     */
    public function init()
    {
        $this->setTemplatePath(
            sprintf('%sskins/%s/', $this->module_path, $this->module_info->skin)
        );
    }

    /**
     * Display item list.
     *
     * @return BaseObject|void
     */
    public function dispMyModuleList()
    {
        $args = new stdClass;
        $args->module_srl = $this->module_srl;
        $args->page = \Context::get('page');
        $output = executeQueryArray('mymodule.getItemList', $args);

        \Context::set('item_list', $output->data);
        \Context::set('page_navigation', $output->page_navigation);
        $this->setTemplateFile('list');
    }
}
```

### Controller Class ({module_name}.controller.php)

```php
<?php

class MyModuleController extends MyModule
{
    /**
     * Initialize the controller.
     *
     * @return void
     */
    public function init()
    {
    }

    /**
     * Insert an item.
     *
     * @return BaseObject|void
     */
    public function procMyModuleInsert()
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

### Model Class ({module_name}.model.php)

```php
<?php

class MyModuleModel extends MyModule
{
    /**
     * Initialize the model.
     *
     * @return void
     */
    public function init()
    {
    }

    /**
     * Get an item by item_srl.
     *
     * @param int $item_srl
     * @return object|null
     */
    public function getItem(int $item_srl): ?object
    {
        $args = new stdClass;
        $args->item_srl = $item_srl;
        $output = executeQuery('mymodule.getItem', $args);
        return $output->data ?? null;
    }
}
```

### Admin View ({module_name}.admin.view.php)

```php
<?php

class MyModuleAdminView extends MyModule
{
    /**
     * Initialize the admin view.
     *
     * @return void
     */
    public function init()
    {
        $this->setTemplatePath(sprintf('%stpl/', $this->module_path));
    }

    /**
     * Display admin list.
     *
     * @return BaseObject|void
     */
    public function dispMyModuleAdminList()
    {
        $output = executeQueryArray('mymodule.getItemList', new stdClass);
        \Context::set('item_list', $output->data);
        $this->setTemplateFile('admin_list');
    }
}
```

### Mobile Class ({module_name}.mobile.php)

```php
<?php

class MyModuleMobile extends MyModuleView
{
    // Extends View class; override methods for mobile-specific behavior.
}
```

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
