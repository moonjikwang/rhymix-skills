# Rhymix Skills

Agent Skills for [Rhymix CMS](https://rhymix.org) extension development.

## Skills

### rhymix-dev

Comprehensive guide for Rhymix CMS extension development — modules, addons, layouts, widgets, and skins. Validates and generates code following the [official documentation](https://rhymix.org/manual).

**Covers:**
- Module development (MVC structure, actions, triggers, routing)
- Database schema and XML query definitions
- Template v1/v2 syntax
- Addon, layout, widget, widgetstyle, skin development
- Coding standards and API reference

## Installation

```bash
npx skills add https://github.com/moonjikwang/rhymix-skills --skill rhymix-dev
```

Or install all skills:

```bash
npx skills add https://github.com/moonjikwang/rhymix-skills
```

## Usage

After installation, the skill is available in Claude Code (and other compatible agents):

- **Auto-trigger**: Mention Rhymix, XE, or work on Rhymix extension code
- **Manual invoke**: `/rhymix-dev`

## Compatibility

Works with any agent that supports the [Agent Skills](https://agentskills.io) standard — Claude Code, Cursor, GitHub Copilot, Gemini CLI, and more.

## License

MIT
