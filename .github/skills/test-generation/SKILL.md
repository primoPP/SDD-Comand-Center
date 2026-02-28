---
name: test-generation
description: Generate comprehensive tests from specifications
version: 5.0.1
project: vscode-copilot-chat
trigger: "#test-generation"
auto-trigger: when creating tests for a batch
related-agents:
  - test
  - implementation
language: en
---

# Test Generation Skill

## Purpose

Generate comprehensive tests from specifications for vscode-copilot-chat.

## Test Types

| Type | Layer | Purpose |
|------|-------|---------|
| Unit | Domain | Test business logic in isolation |
| Integration | Application | Test use case orchestration |
| Component | Infrastructure | Test adapters with real dependencies |
| E2E | API/UI | Test full user flows |

## Workflow

```
1. READ specification (.feature.toon, .acceptance.toon)
2. EXTRACT scenarios from spec
3. MAP scenarios to test cases
4. GENERATE test files
5. VERIFY coverage of all scenarios
```

## From Specification to Tests

```toon
scenarios[3]{id,name,type}:
  SC-001,Create order success,HAPPY_PATH
  SC-002,Below minimum value,ERROR_CASE
  SC-003,Empty cart,ERROR_CASE
```

Generates:

```
test("should create order successfully") { ... }
test("should reject order when below minimum") { ... }
test("should reject order when cart empty") { ... }
```

## Test Naming Convention

```
[should/when] + [expected behavior] + [condition]
```

## Given-When-Then Pattern

```
// Given: Setup preconditions
// When: Execute action
// Then: Verify outcomes
```

## Coverage Requirements

| Scenario Type | Required Tests |
|---------------|----------------|
| HAPPY_PATH | 1 test minimum |
| ERROR_CASE | 1 test per business rule |
| EDGE_CASE | 1 test per boundary |
| SECURITY | 1 test per requirement |

## References

- **Test Patterns**: See `test.prompt.template.md`
- **Related Skills**: `#code-generation`, `#code-review`
- **Related Agents**: `@test`, `@implementation`
