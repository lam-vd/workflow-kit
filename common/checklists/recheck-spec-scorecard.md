# Recheck Spec Scorecard

Use this in Stage 4 for consistent scoring.

## Pre-scoring checks (before rubric)

- [ ] BD EN canonical matches VI + JP collapsible blocks (scope, non-goals, user stories)
- [ ] DDD EN has inline Mermaid: flowchart + happy sequence + error sequence
- [ ] Amendment / changelog version reflected across BD, DDD EN, DDD `.vi.md` (ai-housemaker)
- [ ] No "see other file" for diagrams unless that file contains the diagrams

## Rubric items (0.0 / 0.5 / 1.0)

1. Open Questions resolved
2. Edge cases coverage (>=5)
3. Flow + sequence diagrams (inline in EN DDD)
4. API contract completeness (schema + examples + error codes)
5. Migration + rollback
6. Quantitative test plan
7. Observability plan
8. Security considerations
9. Performance budget
10. Consistency + clarity (BD ↔ DDD; BD EN = VI = JP)

## Severity mapping

- Red: 0.0 and auto-block
- Orange: 0.0 (includes tri-lingual BD drift, missing/deferred diagrams)
- Yellow: 0.5
- Green: no deduction

## Formula

- Standard: sum(10 items) / 10
- If N/A exists: sum(applied items) / applied_count * 10

## Gate

- score >= 8.0 and no Red: pass
- score < 8.0 and no Red: user decides fix or bypass
- any Red: blocked

## Common regressions (ai-housemaker / tri-lingual BD)

| Pattern | Item | Severity |
|---|---|---|
| VI/JP non-goals contradict EN goals | #10 | Orange |
| DDD diagrams only in `.vi.md` placeholder | #3 | Orange |
| vN amendment missing flowchart update | #10 | Yellow |
| `error_key` table incomplete | #4 | Yellow |
