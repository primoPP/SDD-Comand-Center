---
name: dor-epic-validation
description: Validate Definition of Ready for Epics before planning
version: 5.0.1
project: vscode-copilot-chat
trigger: "#dor-epic-validation"
auto-trigger: before creating .feature.toon from epic
related-agents:
  - product-owner
  - planning
language: en
---

# DoR Epic Validation Skill

## Purpose

Validate that an Epic meets all Definition of Ready criteria before entering development planning.

## When to Use

- Before `@product-owner` hands off to `@planning`
- When creating a new `.feature.toon` from business requirements
- At epic refinement sessions

## Validation Workflow

```
1. READ .sdd/definitions/dor-epic.definition.toon
2. FOR EACH category (requisitos_funcionais, requisitos_nao_funcionais, etc.)
   2.1 CHECK each item with required=true
   2.2 RECORD status (PASS/FAIL/NA)
3. IF any required item FAILS
   3.1 LIST missing items
   3.2 STOP and request clarification
4. IF all required items PASS
   4.1 GENERATE DoR status summary
   4.2 PROCEED to @planning
```

## Output Format

### When PASSED
```
✅ DoR EPIC VALIDATION: PASSED

Requisitos Funcionais:     5/5 ✓
Requisitos Não Funcionais: 8/8 ✓
Métricas/Observabilidade:  1/1 ✓
Dependências:              2/2 ✓
Refinamento:               1/1 ✓

Epic is READY for planning.
```

### When FAILED
```
❌ DoR EPIC VALIDATION: BLOCKED

Missing Required Items:
- [ ] arquitetura-definida: Está definido o desenho da arquitetura?
- [ ] appsec-envolvido: O time de AppSec foi envolvido?

Action Required:
1. Define architecture before proceeding
2. Involve AppSec team for security review

Epic is NOT READY. Resolve items above before planning.
```

## Integration with Agents

| Agent | When | Action |
|-------|------|--------|
| `@product-owner` | Before handoff | Run validation, ensure all items pass |
| `@planning` | Before creating spec | Verify DoR was validated |

## Customization

Teams can add items to `.sdd/definitions/dor-epic.definition.toon` but should not remove the minimum required items from the base template.

## References

- **DoR Template**: `bootstrap/definitions/dor-epic.definition.toon`
- **DoR Documentation**: See Confluence DoR page
- **Related Skills**: `#dor-story-validation`, `#dod-validation`
- **Related Agents**: `@product-owner`, `@planning`
