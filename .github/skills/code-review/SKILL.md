---
name: code-review
description: Execute systematic code review following project standards
version: 5.0.1
project: vscode-copilot-chat
trigger: "#code-review"
auto-trigger: during review phase
related-agents:
  - review
  - implementation
language: en
---

# Code Review Skill

## Purpose

Execute systematic code review following vscode-copilot-chat standards.

## Review Checklist

### 1. Specification Compliance
- [ ] Code matches feature spec exactly
- [ ] All business rules implemented
- [ ] All scenarios covered
- [ ] Input validations match constraints

### 2. Architecture Compliance
- [ ] Follows project layer structure
- [ ] Dependencies flow correctly (outer → inner)
- [ ] No circular dependencies
- [ ] Proper separation of concerns

### 3. Code Quality
- [ ] Naming follows project conventions
- [ ] No code duplication (DRY)
- [ ] Single responsibility principle
- [ ] Proper error handling

### 4. Testing
- [ ] Unit tests for business logic
- [ ] Test coverage >= project threshold
- [ ] Edge cases covered

### 5. Security
- [ ] Input validation present
- [ ] No sensitive data in logs
- [ ] Authorization enforced

## Workflow

```
1. READ specification (.feature.toon, .architecture.toon)
2. READ implementation files
3. COMPARE code vs specification
4. EVALUATE against checklist
5. GENERATE review report
```

## Output Format

### Approved
```
✅ Code Review: APPROVED
Checklist: All items passed
Coverage: 87%
```

### Changes Requested
```
⚠️ Code Review: CHANGES REQUESTED
1. [CRITICAL] Specification Mismatch - Line 45
2. [MEDIUM] Missing Test - SC-002 not covered
```

## Severity Levels

| Level | Action |
|-------|--------|
| CRITICAL | Must fix before merge |
| HIGH | Should fix in this PR |
| MEDIUM | Fix in PR or tech debt |
| LOW | Tech debt OK |

## References

- **Review Workflow**: See `@review` agent
- **Related Skills**: `#test-generation`, `#code-generation`
- **Related Agents**: `@review`, `@implementation`
