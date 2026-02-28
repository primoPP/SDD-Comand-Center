---
name: dod-validation
description: Validate Definition of Done before approving a feature
version: 5.0.1
project: vscode-copilot-chat
trigger: "#dod-validation"
auto-trigger: during review phase before approval
related-agents:
  - review
language: en
---

# DoD Validation Skill

## Purpose

Validate that a feature meets all Definition of Done criteria before approval.

## When to Use

- When `@review` evaluates a completed implementation
- Before approving a feature and updating CHANGELOG
- As final quality gate before merge

## Validation Workflow

```
1. READ .sdd/definitions/dod.definition.toon
2. FOR EACH category
   2.1 EVALUATE each checklist item
   2.2 MARK as PASS/FAIL/NA
   2.3 For FAIL items, document reason
3. GENERATE DoD report
4. IF any required item FAILS
   4.1 REJECT with detailed feedback
   4.2 LIST items to fix
5. IF all required items PASS
   5.1 APPROVE feature
   5.2 UPDATE CHANGELOG.md
```

## Output Format

### DoD Report (included in review report)
```markdown
## DoD Validation Report

Feature: Create Order
Review Date: 2026-01-30
Reviewed By: @review

### Requisitos Funcionais (4/4) ✅
- [x] criterios-aceite-atendidos
- [x] objetivo-iniciativa
- [x] regras-negocio-reguladores
- [x] validacao-stakeholders

### Requisitos Não Funcionais (5/5) ✅
- [x] dor-resolvido
- [x] tagueamento-infra
- [x] appsec-validou
- [x] catalogo-servicos
- [x] boas-praticas-bd

### Qualidade (6/7) ⚠️
- [x] sonar-guardrails
- [x] cobertura-80 (85% achieved)
- [x] testes-extras
- [x] debitos-documentados
- [x] debitos-plano-acao
- [x] documentacao
- [~] dlq-mensageria (N/A - no messaging)

### Métricas e Observabilidade (4/4) ✅
- [x] alertas-criados
- [x] runbooks-criados
- [x] dashboards-criados
- [x] alerta-negocio

### Estratégia de Rollout (5/7) ✅
- [~] picpay-lovers (N/A - going all-in)
- [x] percentual-base (100%)
- [x] documentacao-usuario
- [x] weduka-atualizado
- [x] tags-atendimento
- [x] faqs-atualizadas
- [~] campanha-mkt (N/A - no campaign needed)

---

**DoD Status: PASSED ✅**
All required items validated. Feature approved.
```

### When REJECTED
```
❌ DoD VALIDATION: FAILED

Required Items Missing:
1. cobertura-80: Current coverage is 65%, required 80%
2. runbooks-criados: No runbook found for error scenarios

Action Required:
- Increase test coverage to 80%
- Create runbook for payment failure scenarios

Feature NOT approved. Fix items above and request re-review.
```

## Integration with @review

The `@review` agent MUST:
1. Read `.sdd/definitions/dod.definition.toon`
2. Validate each item during review
3. Include DoD report in review output
4. Only approve if all required items pass

## References

- **DoD Template**: `bootstrap/definitions/dod.definition.toon`
- **Related Skills**: `#dor-epic-validation`, `#dor-story-validation`, `#code-review`
- **Related Agents**: `@review`
