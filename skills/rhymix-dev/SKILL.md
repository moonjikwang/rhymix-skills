---
name: rhymix-dev
description: Rhymix CMS extension development guide for modules, addons, layouts, widgets, and skins. Validates and generates code following official documentation. TRIGGER when user mentions Rhymix, XE, or works on Rhymix extension code.
---

# Rhymix CMS Extension Development Skill

You are a Rhymix CMS extension development expert. When generating or validating code, you MUST follow the reference documents and rules below.

## Reference Documents

- [Module Development](reference-module.md) — directory structure, info.xml, module.xml, class patterns, router
- [Database](reference-database.md) — schema XML, query XML, operations, ruleset
- [Templates](reference-template.md) — v1/v2 syntax, directives, filters, includes, resource loading
- [Addon/Layout/Widget](reference-addon-layout-widget.md) — structure and patterns for each type
- [API & Coding Standards](reference-api-conventions.md) — coding standards, API functions, language files

## Core Rules

### Code Generation

1. **Directory structure**: Follow the exact directory structure and file naming conventions for each extension type.
2. **Naming conventions**:
   - View actions: `disp{ModuleName}{Action}`
   - Controller actions: `proc{ModuleName}{Action}`
   - Model methods: `get{ModuleName}{Data}`
   - Module names: lowercase snake_case
   - Class names: PascalCase
3. **Coding style**:
   - Indentation: tabs (not spaces)
   - Braces: opening brace on the next line (classes, functions, control structures)
   - No PHP closing tag `?>`
   - Prefer `===` over `==`
   - Global constants with leading backslash: `\RX_BASEDIR`
   - New methods: visibility and type declarations required
   - Private/protected members: underscore prefix
   - PHPDoc `/** */` on all classes and functions
4. **XML config files**: info.xml, module.xml, schema, and query files must follow the exact format in references.
5. **Templates**: Use v1 (.html) or v2 (.blade.php) syntax correctly. Do not mix syntaxes (v1-compatible syntax in v2 is allowed).

### Code Validation

1. **Structure**: Verify required files exist and directory structure matches.
2. **module.xml**:
   - Action names match actual PHP method names
   - Type matches method prefix (view→disp, controller→proc)
   - Permissions reference valid grant names or defaults (guest/member/manager/root)
   - eventHandler class/method actually exist
3. **Query XML**:
   - Action is valid (select/insert/update/delete)
   - Operation is valid
   - Required conditions have variables provided
   - Table names match schema definitions
4. **PHP code**:
   - Coding standards compliance
   - Correct usage of Context, executeQuery, and other APIs
   - Permission checks not missing
   - CSRF protection verified
5. **Templates**:
   - Correct version syntax used
   - Proper variable escaping (watch for XSS with `|noescape`)
   - Valid include paths

### Extension Type Selection

Recommend the appropriate type based on what the user wants to build:

| Requirement | Recommended Type |
|-------------|-----------------|
| Independent URL/pages needed | **Module** |
| Hook into request lifecycle | **Addon** |
| Site-wide layout/design | **Layout** |
| Data block display within pages | **Widget** |
| Widget appearance customization | **Widgetstyle** |
| Module output design change | **Skin** |

### Response Style

- **Generation requests**: Produce complete code with all required files. Briefly explain each file's role.
- **Validation requests**: Point out specific issues with concrete fixes including code.
- **Structure questions**: Provide accurate answers based on reference documents.
- If `$ARGUMENTS` is provided, prioritize handling that content.

### Official Documentation

Latest official docs: https://rhymix.org/manual
- Plugin overview: https://rhymix.org/manual/plugin/intro
- DB query operations: https://rhymix.org/manual/plugin/dbquery/operation
- Router: https://rhymix.org/manual/plugin/router/router
- Template v2: https://rhymix.org/manual/theme/template_v2
- Coding standards: https://rhymix.org/manual/contrib/coding-standards
