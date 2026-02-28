---
name: architecture-decision
description: Create Architecture Decision Records (ADRs)
version: 5.0.1
project: vscode-copilot-chat
trigger: "#architecture-decision"
auto-trigger: when documenting architectural decisions
related-agents:
  - architecture
  - planning
language: en
---

# Architecture Decision Skill

## Purpose

Create Architecture Decision Records (ADRs) for vscode-copilot-chat.

## When to Create an ADR

- Choosing between technologies
- Defining architectural patterns
- Establishing coding standards
- Making trade-off decisions
- Deviating from conventions

## ADR Template

```markdown
# ADR-[NUMBER]: [TITLE]

## Status
[PROPOSED | ACCEPTED | DEPRECATED | SUPERSEDED by ADR-XXX]

## Date
[YYYY-MM-DD]

## Context
[What is the issue motivating this decision?]

## Decision
[What is the change we're doing?]

## Consequences

### Positive
- [Benefit 1]

### Negative
- [Drawback 1]

## Alternatives Considered
- Option 1: [Why rejected]
- Option 2: [Why rejected]
```

## File Location

```
.github/specs/architecture/ADR-[NUMBER]-[slug].md
```

## Status Lifecycle

```
PROPOSED → ACCEPTED → DEPRECATED
              ↓
         SUPERSEDED by ADR-XXX
```

## Common ADR Topics

| Category | Examples |
|----------|----------|
| Technology | Database, Message broker, Framework |
| Patterns | Architecture style, Error handling, Logging |
| Standards | Code style, API design, Naming |

## References

- **Architecture Contracts**: See `.architecture.toon` specs
- **Related Skills**: `#feature-spec`
- **Related Agents**: `@architecture`, `@planning`
