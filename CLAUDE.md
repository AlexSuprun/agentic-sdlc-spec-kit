# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Project Is

Agentic SDLC Spec Kit is a toolkit for Spec-Driven Development (SDD) — a methodology where specifications drive implementation via AI coding agents. It consists of:

1. **Specify CLI** (`src/specify_cli/`) — A Python CLI (built with Typer/Rich) that bootstraps projects with SDD templates, commands, and AI agent integrations
2. **Templates** (`templates/`) — Markdown templates for specs, plans, tasks, checklists, architecture docs, and agent command files
3. **Extensions** (`extensions/`) — A modular extension system with dual catalogs (official `catalog.json` + community `catalog.community.json`)
4. **Skills** (`src/specify_cli/skills/`) — A package manager for agent skills with registry, evaluation, and auto-discovery

The CLI supports 18+ AI coding agents (Claude, Copilot, Gemini, Cursor, etc.) via the `AGENT_CONFIG` dict in `__init__.py`.

## Build & Development Commands

```bash
# Install dependencies
uv sync --extra test

# Run CLI locally
uv run specify --help
uv run specify init <project-name> --ai claude

# Lint (Python)
uvx ruff check src/

# Lint (Markdown)
npx markdownlint-cli2 '**/*.md'

# Run all tests
uv run pytest

# Run a single test file
uv run pytest tests/test_extensions.py

# Run a single test function
uv run pytest tests/test_extensions.py::TestExtensionManifest::test_valid_manifest -v

# Test template/command changes locally (generates release packages)
./.github/workflows/scripts/create-release-packages.sh v1.0.0
```

## Architecture

### Single-file CLI (`src/specify_cli/__init__.py`, ~4300 lines)

The entire CLI lives in one large file. Key sections:

- **`AGENT_CONFIG` dict** (line ~135) — Single source of truth for all agent metadata. Keys must be the actual CLI executable name (e.g., `"cursor-agent"` not `"cursor"`).
- **`init()` command** — Bootstraps a project: creates directory structure, downloads templates from GitHub releases, generates agent-specific command files, optionally installs skills and git hooks
- **`check()` command** — Verifies required tools are installed
- **`version()` command** — Displays version and system info
- **`main()` entry point** (line ~4359) — Registered as `specify` in `pyproject.toml`

### Extension System (`src/specify_cli/extensions.py`)

Handles installation, removal, and management of modular extensions. Extensions provide additional commands via `extension.yml` manifests.

### Skills System (`src/specify_cli/skills/`)

Five modules: `manifest.py`, `installer.py`, `registry.py`, `evaluator.py`, `discovery.py`. Treats agent skills as versioned software dependencies with a dual registry (skills.sh for discovery, local manifests for installed skills).

### Templates (`templates/commands/`)

Command templates (e.g., `specify.md`, `plan.md`, `tasks.md`) are Markdown files with `{SCRIPT}`, `$ARGUMENTS`, and `__AGENT__` placeholders that get transformed into agent-specific command files during `init`.

### Scripts (`scripts/bash/`, `scripts/powershell/`)

Platform-specific scripts for spec-code synchronization git hooks and agent context updates.

## Key Conventions

- **Python imports**: Always at file top, sorted: stdlib → third-party → local (isort order). No wildcard imports.
- **Version management**: Changes to `__init__.py` require a version bump in `pyproject.toml` and a CHANGELOG.md entry.
- **Agent config keys**: Must match the actual executable name to avoid special-case mappings throughout the codebase.
- **CI matrix**: Tests run on Python 3.11, 3.12, and 3.13.
- **Markdown linting**: Uses markdownlint-cli2 with custom config (`.markdownlint-cli2.jsonc`). Line length (MD013) is disabled. `extensions/` directory is excluded.
