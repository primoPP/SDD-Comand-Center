---
name: dor-story-validation
description: Validate Definition of Ready for Stories/Tasks before implementation
version: 5.0.1
project: vscode-copilot-chat
trigger: "#dor-story-validation"
auto-trigger: before creating batch from story
related-agents:
  - planning
  - architecture
language: en
---

# DoR Story Validation Skill

## Purpose

Validate that a Story or Task meets all Definition of Ready criteria before starting implementation.

## When to Use

- When `@planning` creates batches from a feature
- Before `@implementation` starts working on a batch
- At story refinement sessions

## Validation Workflow

```
1. READ .sdd/definitions/dor-story.definition.toon
2. FOR EACH batch in the feature
   2.1 CHECK each required item
   2.2 RECORD status per batch
3. IF any required item FAILS for a batch
   3.1 MARK batch as NOT READY
   3.2 LIST missing items
4. IF all items PASS
   4.1 INCLUDE DoR status in .feature.toon header
   4.2 PROCEED to implementation
```

## Output Format

### Per-Batch Validation
```
📋 DoR STORY VALIDATION

Batch 1: Order Entity
├── Requisitos Funcionais:     3/3 ✓
├── Requisitos Não Funcionais: 2/2 ✓
├── Métricas/Observabilidade:  3/3 ✓
├── Dependências:              1/1 ✓
└── Refinamento:               1/1 ✓
Status: READY ✅

Batch 2: Payment UseCase
├── Requisitos Funcionais:     2/3 ✗
│   └── Missing: fluxos-definidos
├── Requisitos Não Funcionais: 2/2 ✓
└── ...
Status: NOT READY ❌

Summary: 1/2 batches ready for implementation
```

## Include in Feature Spec

```toon
feature:
  name: Create Order
  dor_status: VALIDATED
  dor_validated_at: 2026-01-30
  dor_validated_by: @planning
```

## Integration with Agents

| Agent | When | Action |
|-------|------|--------|
| `@planning` | Creating batches | Validate each batch against DoR |
| `@implementation` | Before starting | Verify batch has DoR VALIDATED status |

## References

- **DoR Template**: `bootstrap/definitions/dor-story.definition.toon`
- **Related Skills**: `#dor-epic-validation`, `#dod-validation`, `#batch-planning`
- **Related Agents**: `@planning`, `@architecture`
