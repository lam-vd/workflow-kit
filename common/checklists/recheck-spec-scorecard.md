# Recheck Spec Scorecard

Use this in Stage 4 for consistent scoring.

## Rubric items (0.0 / 0.5 / 1.0)

1. Open Questions resolved
2. Edge cases coverage (>=5)
3. Flow + sequence diagrams
4. API contract completeness
5. Migration + rollback
6. Quantitative test plan
7. Observability plan
8. Security considerations
9. Performance budget
10. Consistency + clarity

## Severity mapping

- Red: 0.0 and auto-block
- Orange: 0.0
- Yellow: 0.5
- Green: no deduction

## Formula

- Standard: sum(10 items) / 10
- If N/A exists: sum(applied items) / applied_count * 10

## Gate

- score >= 8.0 and no Red: pass
- score < 8.0 and no Red: user decides fix or bypass
- any Red: blocked
