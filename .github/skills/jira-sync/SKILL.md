---
name: jira-sync
description: Create and update Jira issues from SDD specifications
version: 5.0.1
project: vscode-copilot-chat
trigger: "#jira-sync"
auto-trigger: when syncing batches/features with Jira
type: integration
requires-config: true
language: en
---

# Jira Sync Skill

## Purpose

Create and update Jira issues from SDD specifications for vscode-copilot-chat.

## Configuration Required

```yaml
# .sdd/integrations.yaml
jira:
  project_key: [YOUR_PROJECT_KEY]
  base_url: https://[your-domain].atlassian.net
  default_issue_type: Story
```

## Workflow

```
1. READ batch plan or feature spec
2. CHECK for existing Jira issues
3. CREATE new issues for new batches
4. UPDATE existing issues with progress
5. LINK issues to Epic
6. REPORT sync status
```

## Mapping: SDD → Jira

| SDD Element | Jira Field |
|-------------|------------|
| Feature name | Epic title |
| Batch name | Story title |
| Acceptance criteria | Description |
| Priority | Priority |

## Issue Templates

### Epic (from Feature)
```
Title: [FT-XXX] [Feature Name]
Type: Epic
Description: [From feature spec]
```

### Story (from Batch)
```
Title: [Batch N] [Component Name]
Type: Story
Epic Link: [Parent Epic]
```

## Output Format

```
# Jira Sync Report

| Type | Key | Title | Status |
|------|-----|-------|--------|
| Epic | PROJ-123 | Create Order | Created ✅ |
| Story | PROJ-124 | Batch 1 | Created ✅ |
```

## Error Handling

| Error | Resolution |
|-------|------------|
| 401 Unauthorized | Check API token |
| 404 Project not found | Verify project_key |

## References

- **Related Skills**: `#batch-planning`, `#confluence-sync`
- **Related Agents**: `@orchestrator`
