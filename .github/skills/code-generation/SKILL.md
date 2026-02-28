---
name: code-generation
description: Generate production code from TOON specifications
version: 5.0.1
project: vscode-copilot-chat
trigger: "#code-generation"
auto-trigger: when implementing a batch from specification
related-agents:
  - implementation
  - architecture
language: en
---

# Code Generation Skill

## Purpose

Generate production code from TOON specifications for vscode-copilot-chat.

## Inputs Required

| Input | Source | Description |
|-------|--------|-------------|
| Feature Spec | `.feature.toon` | Business requirements |
| Architecture Spec | `.architecture.toon` | Technical contracts |
| Batch Plan | Batch planning output | What to implement |

## Workflow

```
1. READ specification files
2. IDENTIFY target layer (domain/application/infrastructure)
3. APPLY project patterns and conventions
4. GENERATE code files
5. GENERATE corresponding tests
6. VALIDATE against specification
```

## Layer-Specific Generation

### Domain Layer
- Entity class with properties
- Value objects with validation
- Domain events (if applicable)
- Repository interface (port)

### Application Layer
- Use case / Handler class
- Input/Output DTOs
- Port interfaces

### Infrastructure Layer
- Repository implementation
- External service clients
- Message handlers

## From Specification

```toon
command:
  name: CreateOrderCommand
  inputs[2]{field,type,constraints}:
    customerId,UUID,required
    items,List<OrderItem>,"required,min:1"
```

Generates:
- Command/Input DTO
- Use Case with validation
- Repository Port

## Quality Checks

- [ ] All inputs from spec present
- [ ] All business rules implemented
- [ ] All validation constraints applied
- [ ] Error handling in place
- [ ] Tests cover all scenarios

## References

- **Prompts**: Use `domain.prompt.template.md`, `use-case.prompt.template.md`
- **Related Skills**: `#batch-planning`, `#test-generation`, `#code-review`
- **Related Agents**: `@implementation`, `@architecture`
