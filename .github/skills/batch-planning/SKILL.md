---
name: batch-planning
description: Divide features into implementation batches using Three Pillars
version: 5.0.1
project: vscode-copilot-chat
trigger: "#batch-planning"
auto-trigger: when breaking features into batches
related-agents:
  - planning
  - orchestrator
language: en
---

# Batch Planning Skill

## Purpose

Divide features into manageable batches following the Three Pillars principle.

## Three Pillars Principle

Each batch MUST generate exactly three files:

```
┌─────────────────────────────────────────────────────────────┐
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │   DOMAIN    │  │  USE CASE   │  │    TEST     │         │
│  │   ENTITY    │  │   (Prompt)  │  │  SPEC FILE  │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
└─────────────────────────────────────────────────────────────┘
```

## Workflow

```
1. READ feature specification
2. IDENTIFY domain entities needed
3. IDENTIFY use cases/commands
4. MAP dependencies between components
5. CREATE batch sequence (dependency order)
6. GENERATE batch plan
```

## Batch Sequencing Rules

```
1. Domain Entities (no dependencies)
2. Value Objects (depend on entities)
3. Domain Services (depend on entities + VOs)
4. Use Cases (depend on domain layer)
5. Adapters (depend on use cases)
6. Controllers/Handlers (depend on adapters)
```

## Output Format

```markdown
# Batch Plan: [Feature Name]

## Batch 1: [Entity Name] Entity
**Complexity**: LOW | MEDIUM | HIGH
**Dependencies**: None

Files to Generate:
1. Domain entity
2. Core use case  
3. Unit tests

## Batch 2: [UseCase Name] Use Case
**Dependencies**: Batch 1

Files to Generate:
1. Value object
2. Application service
3. Tests
```

## Complexity Guidelines

| Complexity | Lines | Time |
|------------|-------|------|
| LOW | <200 | <30min |
| MEDIUM | 200-500 | 30-60min |
| HIGH | 500+ | 60+min |

## References

- **Three Pillars**: See `copilot-bootstrap-modular.prompt.md` Section 9
- **Related Skills**: `#feature-spec`, `#code-generation`
- **Related Agents**: `@planning`, `@orchestrator`
