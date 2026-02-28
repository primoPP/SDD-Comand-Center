---
name: doa-metrics
description: Collect and report Definition of Agility metrics
version: 5.0.1
project: vscode-copilot-chat
trigger: "#doa-metrics"
auto-trigger: after feature completion
related-agents:
  - orchestrator
language: en
---

# DoA Metrics Skill

## Purpose

Collect and report Definition of Agility metrics to track team agility health.

## When to Use

- After `@orchestrator` completes a feature workflow
- During sprint retrospectives
- For team health assessments

## Metrics Collected

| Metric | Formula | Target | SDD Source |
|--------|---------|--------|------------|
| **Cycle Time** | Review approved - Feature.toon created | < 5 days/batch | Git timestamps |
| **Batch Size** | Lines of code per batch | < 500 lines | Implementation diff |
| **First-Time Approval** | % batches approved on first review | > 80% | Review history |
| **DoR Compliance** | % features with complete DoR | 100% | DoR validation logs |

## Workflow

```
1. READ .sdd/definitions/doa.definition.toon
2. COLLECT metrics for completed feature
   2.1 Calculate cycle time from git history
   2.2 Count lines changed per batch
   2.3 Check if first review was approval
   2.4 Verify DoR was validated
3. COMPARE against targets
4. GENERATE metrics report
```

## Output Format

### Feature Metrics Report
```markdown
## DoA Metrics Report

Feature: Create Order
Completed: 2026-01-30

### Cycle Time
| Batch | Start | End | Duration | Target |
|-------|-------|-----|----------|--------|
| Batch 1 | Jan 25 | Jan 27 | 2 days | < 5 ✅ |
| Batch 2 | Jan 27 | Jan 29 | 2 days | < 5 ✅ |
| Batch 3 | Jan 29 | Jan 30 | 1 day | < 5 ✅ |
**Average: 1.7 days** ✅

### Batch Size
| Batch | Lines | Target |
|-------|-------|--------|
| Batch 1 | 245 | < 500 ✅ |
| Batch 2 | 312 | < 500 ✅ |
| Batch 3 | 189 | < 500 ✅ |
**Average: 249 lines** ✅

### Quality Metrics
- First-Time Approval: 3/3 batches (100%) ✅
- DoR Compliance: Validated ✅

### SDD Contribution to DoA
✅ Entregas com propósito claro (specs in .toon)
✅ Gestão de WIP (Three Pillars enforced)
✅ Métricas de velocidade (cycle time tracked)
```

## Trend Tracking

Store metrics in `.sdd/metrics/` for trend analysis:

```
.sdd/metrics/
├── 2026-01-feature-create-order.md
├── 2026-01-feature-payment-gateway.md
└── summary.md (aggregated metrics)
```

## Integration with Agents

| Agent | When | Action |
|-------|------|--------|
| `@orchestrator` | Feature complete | Generate metrics report |
| `@bootstrap` | On request | Show metrics summary |

## DoA Checklist Validation

Besides metrics, validate qualitative items:

```
DoA Qualitative Check:
- [x] Entregas com propósitos claros? (specs define objectives)
- [x] Decisões baseadas em dados? (metrics tracked)
- [x] DoR/DoD seguidos? (gates validated)
- [x] Gestão de WIP? (batch size < limit)
- [x] Métricas de processo? (cycle time, not individual)
```

## References

- **DoA Template**: `bootstrap/definitions/doa.definition.toon`
- **Related Skills**: `#dor-epic-validation`, `#dor-story-validation`, `#dod-validation`
- **Related Agents**: `@orchestrator`
