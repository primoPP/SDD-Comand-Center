---
name: feature-spec
description: Create structured feature specifications in TOON format
version: 5.0.1
project: vscode-copilot-chat
trigger: "#feature-spec"
auto-trigger: when creating new feature specifications
related-agents:
  - planning
  - product-owner
language: en
---

# Feature Spec Skill

## Purpose

Create structured feature specifications in TOON format for vscode-copilot-chat.

## Inputs Required

| Input | Description | Example |
|-------|-------------|---------|
| Feature Name | Business capability | "Create Order" |
| Priority | HIGH, MEDIUM, LOW | HIGH |
| Requirements | What the feature should do | User story or BRD excerpt |

## Workflow

```
1. ANALYZE requirements provided
2. IDENTIFY command/query pattern
3. DEFINE inputs with types and constraints
4. EXTRACT business rules
5. DESIGN scenarios (happy path + edge cases)
6. GENERATE .feature.toon file
7. VALIDATE with #toon-validation
```

## Output Template

```toon
# TOON v3.0 – Feature Specification
# Generated for: vscode-copilot-chat
# Date: [CURRENT_DATE]

feature:
  name: [FeatureName]
  version: 1.0.0

meta:
  id: FT-[SEQUENCE]
  priority: [PRIORITY]
  domain: VS Code Copilot Chat Extension

command:
  name: [CommandName]
  description: [Brief description]
  inputs[N]{field,type,constraints}:
    [field1],[type1],[constraints1]
    [field2],[type2],[constraints2]
  output:
    type: [OutputType]
    description: [What is returned]

business-rules[N]{id,name,rule}:
  BR-001,[RuleName],[Rule description]
  BR-002,[RuleName],[Rule description]

scenarios[N]{id,name,type,description}:
  SC-001,Happy path,HAPPY_PATH,[Description]
  SC-002,[ErrorCase],ERROR_CASE,[Description]
  SC-003,[EdgeCase],EDGE_CASE,[Description]
```

## Scenario Types

| Type | When to Use | Minimum |
|------|-------------|---------|
| HAPPY_PATH | Success flow | 1 required |
| ERROR_CASE | Business rule violations | 1 per rule |
| EDGE_CASE | Boundary conditions | Optional |
| SECURITY | Auth/authz scenarios | If applicable |

## File Location

```
.github/specs/features/FT-[ID].[feature-name].feature.toon
```

## References

- **TOON Spec**: See `copilot-bootstrap-modular.prompt.md` Section 8
- **Related Skills**: `#toon-validation`, `#batch-planning`
- **Related Agents**: `@planning`, `@product-owner`
