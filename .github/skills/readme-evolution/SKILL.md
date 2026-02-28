---
name: readme-evolution
description: Manage README structure and CHANGELOG entries
version: 5.0.1
project: vscode-copilot-chat
trigger: "#readme-evolution"
auto-trigger: after feature completion or initial setup
type: integration
related-agents:
  - review
  - bootstrap
language: en
---

# README Evolution Skill

## Purpose

Manage documentation evolution for vscode-copilot-chat:
- **README.md**: Structured by @bootstrap (initial setup + major updates)
- **CHANGELOG.md**: Updated by @review (feature entries)

## Documentation Responsibility Split

```
╔════════════════════════════════════════════════════════════════╗
║  RESPONSIBILITY MATRIX                                         ║
╠═══════════════════╦═══════════════╦════════════════════════════╣
║  File             ║  Agent        ║  When                      ║
╠═══════════════════╬═══════════════╬════════════════════════════╣
║  README.md        ║  @bootstrap   ║  Initial specialization    ║
║                   ║               ║  Major structural changes  ║
╠═══════════════════╬═══════════════╬════════════════════════════╣
║  CHANGELOG.md     ║  @review      ║  After each feature        ║
║                   ║               ║  approved                  ║
╚═══════════════════╩═══════════════╩════════════════════════════╝
```

## README Structure (Confluence-Ready)

README should be rich enough to serve as official project documentation:

```markdown
# vscode-copilot-chat

> [Clear business context description]

## 📋 Table of Contents
- About | Tech Stack | Prerequisites | Installation | Execution
- Tests | API | Architecture | SDD | Contributing | Changelog

## 📖 About
[2-3 paragraphs of business context]

### Main Features
[Business capabilities list]

### Domain Entities
[Entity table with descriptions]

## 🛠 Tech Stack
[Technology table]

## 🏗 Architecture
[Architecture diagram or description]

## 🤖 SDD Bootstrap
[SDD section with agents and commands]

## 📝 Changelog
See [CHANGELOG.md](CHANGELOG.md) for version history.
```

## CHANGELOG Format

```markdown
# Changelog

Based on [Keep a Changelog](https://keepachangelog.com).

## [Unreleased]

### Added
- **Feature Name**: Description

### Changed
- **Component**: What changed

### Fixed
- **Bug**: What was fixed
```

## When README is Updated

| Trigger | Action |
|---------|--------|
| First @bootstrap | Structure full README |
| @bootstrap evolve | Update SDD section only |
| Major architecture change | Update Architecture section |

## When CHANGELOG is Updated

| Trigger | Action |
|---------|--------|
| @review approves feature | Add entry to [Unreleased] |
| Release created | Move [Unreleased] to version |

## References

- **README Structure**: Defined by `@bootstrap`
- **CHANGELOG Entries**: Managed by `@review`
- **Related Skills**: `#code-review`
- **Related Agents**: `@bootstrap`, `@review`

- **README Template**: See `@bootstrap` Initial Setup
- **Related Skills**: `#code-review`, `#confluence-sync`
- **Related Agents**: `@review`, `@bootstrap`
