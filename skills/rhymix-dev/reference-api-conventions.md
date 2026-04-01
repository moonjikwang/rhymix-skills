# Rhymix API and Coding Standards Reference

## Coding Standards

### Encoding and Line Endings
- UTF-8 without BOM
- Unix line endings (LF)

### Indentation
- Default: 1 tab character (not spaces)
- Exception: .md, .yml, .json files use 2-4 spaces
- Never mix tabs and spaces

### PHP Files
- Omit closing `?>` tag in PHP-only files

### Brace Placement
- Classes, functions, control structures: opening brace on the NEXT line
- Single-statement blocks still require braces
- Closures allow same-line braces: `function($bar) { return $bar + 1; };`

```php
class MyClass
{
    public function myMethod(): void
    {
        if ($condition)
        {
            // code
        }
        else
        {
            // code
        }

        foreach ($items as $item)
        {
            // code
        }
    }
}
```

### Function Calls
- No space between function name and parentheses
- No space inside parentheses
- One space after commas separating arguments

### Arrays
- One space after comma separator
- Multi-line arrays: trailing comma after final element (PHP only)
- JavaScript/JSON: no trailing commas

### Operators
- Spaces before and after comparison operators
- One space after keywords (`if`, `for`, etc.)

### Visibility and Type Declarations
- New methods: MUST declare `public`/`protected`/`private`
- New methods: MUST declare parameter types and return types
- Private/protected members: underscore prefix (`_privateMethod()`, `$_privateVar`)

### Comparisons
- Prefer `===` over `==`
- Use `intval()`, `strval()`, or casting when types differ

### Global Constants
- Always use leading backslash: `\RX_BASEDIR`, `\RX_CLIENT_IP`

### Error Handling
- Target zero errors at all reporting levels
- Handle `E_NOTICE`, `E_DEPRECATED`, etc.

### PHPDoc
- All classes and functions require `/** */` documentation
- Primary language: English (Korean acceptable)
- Place comments above related code

### Frontend Code
- Source code must remain uncompressed/unobfuscated
- External libraries may be minified

### PHP Version Compatibility
- Union types (PHP 8.0+) prohibited in core code
- Code must work across all supported PHP versions

### Undefined Rules
- Follow PSR-1 and PSR-12

## Key API Functions

### Module Instance Factories

```php
getController('module_name')       // Controller
getModel('module_name')            // Model
getView('module_name')             // View
getAdminController('module_name')  // Admin controller
getAdminModel('module_name')       // Admin model
getAdminView('module_name')        // Admin view
getAPI('module_name')              // API
getMobile('module_name')           // Mobile
getClass('module_name')            // Class
```

### Database

```php
executeQuery('module.query_id', $args)       // Single result
executeQueryArray('module.query_id', $args)  // Array result
getNextSequence()                             // Unique serial number
```

### URL Generation

```php
getUrl('key1', 'val1', 'key2', 'val2')
getNotEncodedUrl('', 'mid', 'board', 'act', 'dispBoardWrite')
```

### Context (Global State)

```php
\Context::set('key', $value)
\Context::get('key')
\Context::getRequestVars()
\Context::getLangType()
\Context::getLang('lang_key')
\Context::loadFile([$path, $type, $media, $index])
\Context::addJsFile($path)
\Context::addCssFile($path)
\Context::get('logged_info')         // Current user info
\Context::get('is_logged')           // Login status
\Context::get('site_module_info')    // Current site info
\Context::getResponseMethod()        // 'HTML', 'JSON', 'XMLRPC'
```

### Utility Functions

```php
config('key')                       // Read system config
lang('lang_key')                    // Get language string
escape($string)                     // HTML escape
toBool($val)                        // Convert to boolean
isCrawler()                         // Detect search engine bots
zdate($date, 'Y-m-d H:i')          // Format Rhymix date string
```

### Triggers

```php
ModuleHandler::triggerCall('trigger_name', 'before|after', $obj)
```

### Exception Classes

```php
throw new \Rhymix\Framework\Exceptions\NotPermitted;
throw new \Rhymix\Framework\Exceptions\InvalidRequest;
throw new \Rhymix\Framework\Exceptions\TargetNotFound;
```

## Language Files (lang/{lang_code}.php)

```php
<?php
$lang->board = '게시판';
$lang->cmd_write = '글쓰기';
$lang->msg_success = '성공적으로 처리되었습니다.';
```

### Access Methods

- Template: `{$lang->board}` or `@lang('board')`
- PHP: `lang('board')` or `\Context::getLang('board')`

### Supported Language Codes

`ko`, `en`, `ja`, `zh-CN`, `zh-TW`, `vi`, `es`, `ru`, `fr`, `tr`

## Commit Message Conventions

### English
- Infinitive verb form, present tense, imperative voice
- Examples: "Delete unnecessary condition", "Fix #1234"
- Avoid: 3rd person singular, past tense

### Korean
- Format: "Where" + "What" + "How"
- Example: "크롬 최신 버전에서 스크립트 오류 해결"
