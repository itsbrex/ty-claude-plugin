# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Claude Code plugin integrating **ty** (Astral's extremely fast Python type checker and LSP written in Rust) with Claude Code.

**Important**: Always use "ty" (lowercase), not "Ty" or "TY".

**Attribution**: Plugin by Brian Roach. ty by [Astral](https://astral.sh).

## Architecture

Monorepo with `/ty-lsp-plugin/` as the distributable plugin directory.

**Critical plugin structure rule**: Plugin manifest files (`plugin.json` and `marketplace.json`) go inside `.claude-plugin/`. The `.lsp.json` and other files must be at the plugin root level.

```
ty-lsp-plugin/
├── .claude-plugin/
│   ├── plugin.json              # Plugin manifest
│   └── marketplace.json         # Local marketplace config
├── .lsp.json                    # LSP config (at plugin root)
├── verify-setup.sh              # Prerequisite verification
└── test/example.py              # Test file with intentional type errors
```

## Development Commands

```bash
# Verify prerequisites (uvx, ty, plugin structure)
./ty-lsp-plugin/verify-setup.sh

# Test plugin locally
claude --plugin-dir ./ty-lsp-plugin

# Test ty type checking directly
uvx ty@latest check ty-lsp-plugin/test/example.py
```

## Marketplace Configuration

The `marketplace.json` (in `.claude-plugin/`) enables local plugin installation. It references plugins by their location:

```json
{
  "name": "ty-lsp-local",
  "description": "Local marketplace for ty LSP plugin",
  "owner": "brianroach/ty-lsp-local",
  "plugins": [
    {
      "name": "ty-lsp",
      "source": {
        "source": "directory",
        "path": "."
      }
    }
  ]
}
```

**Key fields**:
- `owner`: Required marketplace owner in "owner/repo" format
- `plugins[].source`: Specifies plugin location (current directory = `"."`)
- Plugin metadata (description, version, author) belongs in `plugin.json`, not here

To use this plugin locally, add to `~/.claude/settings.json`:

```json
{
  "enabledPlugins": {
    "ty-lsp@ty-lsp-local": true
  },
  "extraKnownMarketplaces": {
    "ty-lsp-local": {
      "source": {
        "source": "directory",
        "path": "/path/to/ty-claude-plugin/ty-lsp-plugin"
      }
    }
  }
}
```

## LSP Configuration

The `.lsp.json` configures how Claude Code launches ty:

```json
{
  "python": {
    "command": "uvx",
    "args": ["ty@latest", "server"],
    "extensionToLanguage": { ".py": "python", ".pyi": "python" }
  }
}
```

**Key points**:
- The server command is `server`, NOT `servern` (common typo)
- `uvx` auto-installs ty; `ty@latest` ensures newest version

## Testing

`test/example.py` has intentional type errors:
- Line 12: String `"5"` passed to `int` parameter
- Line 15: Int `42` passed to `str` parameter

Always run `verify-setup.sh` after modifying LSP configuration.

## Editing Guidelines

- **plugin.json**: Keep attribution clear (ty by Astral, plugin by Brian Roach)
- **Version**: Follows semver in `plugin.json` (current: 1.0.0)
- **Adding tests**: Add Python files to `ty-lsp-plugin/test/` with type annotations and intentional errors
