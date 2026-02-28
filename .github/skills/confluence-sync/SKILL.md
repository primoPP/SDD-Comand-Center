---
name: confluence-sync
description: Synchronize SDD specifications and ADRs with Confluence
version: 5.0.1
project: vscode-copilot-chat
trigger: "#confluence-sync"
auto-trigger: when syncing documentation to Confluence
type: integration
requires-config: true
language: en
---

# Confluence Sync Skill

## Purpose

Synchronize SDD specifications and ADRs with Confluence for vscode-copilot-chat.

## Configuration Required

```yaml
# .sdd/integrations.yaml
confluence:
  base_url: https://[your-domain].atlassian.net/wiki
  space_key: [YOUR_SPACE_KEY]
  parent_page_id: [OPTIONAL]
```

## Workflow

```
1. IDENTIFY files to sync
2. CHECK for existing pages
3. CONVERT format (TOON/MD → Confluence)
4. CREATE or UPDATE pages
5. MAINTAIN page hierarchy
```

## Sync Patterns

```yaml
sync_patterns:
  - ".github/specs/**/*.toon"
  - ".github/specs/architecture/ADR-*.md"
  - ".sdd/business-context.md"
  - ".sdd/domain-vocabulary.md"
```

## Page Structure

```
[Space Home]
└── vscode-copilot-chat Documentation
    ├── 📋 Specifications
    │   ├── Features
    │   └── Architecture (ADRs)
    └── 📖 Domain Context
```

## Format Conversion

TOON tabular arrays → HTML tables
Markdown → Confluence format

## Sync Modes

| Mode | Behavior |
|------|----------|
| `create` | Only new pages |
| `update` | Update existing |
| `full` | Create + Update |
| `dry-run` | Preview only |

## Labels Applied

| Content | Labels |
|---------|--------|
| Feature spec | `sdd`, `feature` |
| ADR | `sdd`, `adr` |
| Domain context | `sdd`, `domain` |

## References

- **Related Skills**: `#jira-sync`, `#architecture-decision`
- **Related Agents**: `@orchestrator`
