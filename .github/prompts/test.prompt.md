---
name: test
description: Template for test infrastructure and patterns
version: 5.0.1
project: vscode-copilot-chat
agent: test
language: en
---

# Test Prompt Template

## Purpose

Define test infrastructure and patterns for the project.

## Project Context

- **Project**: vscode-copilot-chat
- **Domain**: VS Code Copilot Chat Extension — AI-powered coding assistant providing conversational AI, inline editing, agent mode, language model tools, and MCP integration
- **Language**: TypeScript
- **Framework**: VS Code Extension API (vscode ^1.110.0)

## Technical Context

- **Test Framework**: Vitest (unit tests: `*.spec.ts` / `*.spec.tsx`)
- **Test Runner**: Vitest CLI for unit tests; VS Code Extension Host for integration tests; `script/simulate.sh` for simulation tests (`*.stest.ts`)
- **Assertions**: Vitest `expect` API (BDD-style: `expect(x).toBe(y)`, `toEqual`, `toContain`, etc.)
- **Mocking**: Vitest `vi.fn()`, `vi.mock()`, `vi.spyOn()`; manual stubs for VS Code API services
- **Test Data**: Inline fixtures, factory functions, test scenario files in `test/scenarios/`
- **Test Execution**: `npm run test:unit` (unit), `npm run test:extension` (integration), `npm run simulate` (simulation)
- **Code Style**: ESLint with `@typescript-eslint`, tab indentation — `npx eslint .`
- **Coverage**: V8 via Vitest (`--coverage`)

## Blackbox-First Principle

Tests MUST follow the blackbox testing principle:
- Focus on **inputs and outputs** without knowledge of internals
- Verify **behavior from external perspective**
- Derive tests from **specifications and acceptance criteria**

### Test Prioritization

1. **Functional/Integration Tests (PRIMARY)**
   - Test through public interfaces
   - Validate acceptance scenarios
   - Focus on entry points

2. **Unit Tests (SECONDARY)**
   - ONLY for complex algorithms
   - ONLY for critical business logic
   - NOT for simple CRUD or mapping

## Test Directory Structure

```
test/
├── base/                 # Base test utilities and helpers
├── e2e/                  # End-to-end / CLI simulation tests (*.stest.ts)
├── inline/               # Inline chat/edit test suites
├── intent/               # Intent detection tests
├── outcome/              # Outcome evaluation tests
├── prompts/              # Prompt-related tests
├── scenarios/            # Scenario fixtures for simulation tests
├── simulation/           # Simulation test infrastructure
└── codeMapper/           # Code mapping test suites

src/extension/**/test/    # Co-located unit tests (*.spec.ts)
src/platform/**/test/     # Platform service unit tests (*.spec.ts)
```

## Test Patterns

### Integration Test Pattern

```
// Pattern for integration tests (language-agnostic)

TestClass: Create<Entity>IntegrationTest
├── Setup
│   ├── Initialize test container/database
│   ├── Set up test data
│   └── Configure mocks for external services
├── Test Cases (from acceptance scenarios)
│   ├── SC-001: test_happy_path
│   ├── SC-002: test_error_case_1
│   └── SC-003: test_error_case_2
├── Assertions
│   ├── Verify HTTP response status
│   ├── Verify response body
│   ├── Verify database state
│   └── Verify events published
└── Cleanup
    └── Reset test data between tests
```

### Unit Test Pattern (When Justified)

```
// Pattern for unit tests (language-agnostic)

TestClass: ComplexCalculationTest
├── Setup
│   └── Create instance with mocked dependencies
├── Test Cases (edge cases only)
│   ├── test_edge_case_1
│   ├── test_boundary_value
│   └── test_error_condition
└── Assertions
    └── Verify calculation results
```

## Testability Criteria

### When to Test

| Should Test | Test Type |
|-------------|-----------|
| Entry points (controllers, handlers) | Functional/Integration |
| Business workflows | Functional/Integration |
| Complex algorithms | Unit (if in strategy) |

### When NOT to Test

| Should NOT Test | Reason |
|-----------------|--------|
| Simple data containers | No behavior |
| Enums without methods | Just values |
| Generated code | Framework responsibility |
| Simple delegation | Covered by integration |

## Test Commands

```bash
# Run unit tests
npm run test:unit

# Run VS Code extension integration tests
npm run test:extension

# Run simulation tests (scenario-based)
npm run simulate

# Run simulation with specific grep pattern
npm run simulate -- --grep=@pattern --verbose

# Check code style
npx eslint .
```

## Constraints

- MUST follow project test patterns exactly
- MUST NOT introduce new test frameworks
- MUST NOT modify production code to fix tests
- MUST NOT leave skipped or ignored tests
- Generated tests MUST be indistinguishable from manual tests

## Validation Checklist

Before completing:
- [ ] Tests compile successfully
- [ ] All tests pass
- [ ] Tests follow project patterns exactly
- [ ] No new frameworks introduced
- [ ] Full test suite still passes
- [ ] Code style validation passes
