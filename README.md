# Dev Doc Management Skill

A Claude Code skill for structured documentation management in AI-assisted development projects.

## What It Solves

Modern AI-assisted projects often use multiple plugins simultaneously (bmad, superpowers, oh-my-claudecode, etc.). Each plugin injects its own doc path conventions into every conversation, causing AI agents to scatter files across inconsistent locations.

This skill provides a battle-tested strategy to unify documentation regardless of which plugins are active.

## Core Principles

- **Two-zone separation**: `dev_docs/` for developer/AI collaboration, `run_docs/` for users and ops
- **Doc-routing rule file**: A `.claude/rules/doc-routing.md` that overrides all plugin defaults — project rules beat plugin hooks
- **Update-resilient**: Only change project files (`CLAUDE.md`, `.claude/rules/`, plugin project-level configs), never plugin cache files
- **Minimal disclosure**: CLAUDE.md as navigation entry only, details in linked docs
- **Progressive disclosure**: Load docs on demand, not all at once

## Key Features

| Feature | Description |
|---------|-------------|
| Multi-plugin conflict resolution | Resolves path conflicts between bmad, superpowers, omc |
| Two-zone directory template | Clear separation by audience |
| doc-routing.md template | Ready-to-use override rule file |
| Update-resistant strategy | Survives plugin upgrades |
| Global vs project rules | `~/.claude/CLAUDE.md` for cross-project defaults |

## Usage

Install as a Claude Code skill and invoke with `/docs-management` or reference in your project's CLAUDE.md.

See `SKILL.md` for complete guidelines and templates.
