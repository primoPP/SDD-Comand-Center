---
name: toon-validation
description: Validate TOON v3.0 specification syntax and structure
version: 5.0.1
project: vscode-copilot-chat
trigger: "#toon-validation"
auto-trigger: when generating .toon files
related-agents:
  - planning
  - architecture
language: en
---

# TOON Validation Skill

## Purpose

Validate TOON v3.0 specification syntax and structure for vscode-copilot-chat.

## Validation Rules

### 1. Structure Validation

```
✅ Required sections for .feature.toon:
   - feature: (name, version)
   - meta: (id, priority)
   - command: or query: (inputs, output)
   - scenarios: (at least 1)

✅ Required sections for .acceptance.toon:
   - acceptance: (feature, version)
   - criteria: (at least 1 with given/when/then)

✅ Required sections for .architecture.toon:
   - architecture: (component, version)
   - contracts: (at least 1 interface definition)
```

### 2. Syntax Rules

| Rule | Valid | Invalid |
|------|-------|---------|
| Indentation | 2 spaces | tabs, 4 spaces |
| Key-value | `key: value` | `key:value`, `key :value` |
| Tabular | `key[N]{col1,col2}:` | `key[N]:`, `key{col}:` |
| Comments | `# comment` | `// comment` |
| Quoting | Only when needed | Always quoted |

### 3. Tabular Array Validation

```toon
# Valid tabular array
inputs[2]{field,type,constraints}:
  customerId,UUID,required
  amount,Decimal,"required,min:0"

# Invalid - row count mismatch
inputs[3]{field,type}:
  customerId,UUID
  amount,Decimal
# Error: Declared 3 rows, found 2
```

## Workflow

```
1. READ the .toon file
2. VALIDATE structure (required sections)
3. VALIDATE syntax (indentation, formatting)
4. VALIDATE tabular arrays (row counts, column counts)
5. VALIDATE references (IDs, cross-references)
6. REPORT results
```

## Output Format

### Valid File
```
✅ TOON Validation Passed

File: .github/specs/features/FT-001.feature.toon
- Structure: ✓ All required sections present
- Syntax: ✓ Valid TOON v3.0 format
- Tabular: ✓ 3 arrays validated (12 rows total)
- References: ✓ All IDs unique
```

### Invalid File
```
❌ TOON Validation Failed

File: .github/specs/features/FT-001.feature.toon

Errors:
1. Line 5: Invalid indentation (found 4 spaces, expected 2)
2. Line 12: Tabular array row count mismatch (declared 3, found 2)
3. Line 18: Missing required section 'scenarios'

Suggestions:
- Use 2 spaces for indentation
- Update array declaration to match actual rows
- Add scenarios section with at least one scenario
```

## References

- **TOON Spec**: See `copilot-bootstrap-modular.prompt.md` Section 8
- **Related Skills**: `#feature-spec`, `#batch-planning`
- **Related Agents**: `@planning`, `@architecture`
