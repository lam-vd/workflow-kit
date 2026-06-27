---
mode: ask
description: "Stage 6 of the Senior Workflow — Decompose a FINAL-locked spec into prioritized sub-tasks (each ≤ 4 hours) with measurable Definition of Done. Order: P0 schema/contract → P1 core/high-risk → P2 API/integration → P3 UI/glue → P4 polish. Marks parallel-able tasks. Each sub-task has: title, priority, estimate, risk emoji, dependencies, parallel flag, DoD. 🟠/🔴 sub-tasks must include rollback strategy in DoD. Warns if total > 5 days (consider multiple PRs)."
---

You are at **Stage 6: Task Breakdown**.

> **VI**: Bước 6 — Chia nhỏ task thành các sub-task (≤4h mỗi cái), có Definition of Done rõ ràng.

## Precondition / Điều kiện
- Spec is `FINAL` (verify in file header). / Spec đã là FINAL.
- If not → STOP, ask user to run `/check-spec`. / Nếu chưa → yêu cầu chạy `/check-spec`.

## Task / Nhiệm vụ

Read the DDD and decompose into sub-tasks following the rules below.

**VI**: Đọc DDD và chia thành các sub-task theo quy tắc bên dưới.

### Decomposition rules / Quy tắc chia task
1. **Each sub-task ≤ 4h work.** Larger → split further.
2. **Each sub-task has a measurable Definition of Done.**
3. **Priority order**:
   - **P0** — Schema / migration / contract changes (do first; easiest to roll back when wrong).
   - **P1** — Core business logic (high-risk, do early to leave buffer).
   - **P2** — API / integration layers.
   - **P3** — UI / glue code.
   - **P4** — Polish (logs, metrics, docs, error messages).
4. **Mark parallel-able**: which sub-tasks can be picked up in parallel.
5. **Risk per sub-task**: 🟢 / 🟡 / 🟠 / 🔴.

## Output template

```markdown
# 📋 Task Breakdown — <feature name>
**Source spec**: docs/ddd/<file>.md
**Total estimate**: <sum> hours

## Sub-tasks

| # | Title | Priority | Est | Risk | Depends on | Parallel-able | Definition of Done |
|---|---|---|---|---|---|---|---|
| 1 | Add column `xxx` to table `yyy` | P0 | 2h | 🟠 | — | No | Up/down migration runs locally + rollback verified |
| 2 | Create `XxxRepo` repository | P1 | 3h | 🟢 | 1 | No | Unit tests cover happy + 2 edge cases; coverage ≥85% |
| 3 | Implement `DoXxx` service | P1 | 4h | 🟡 | 2 | No | Integration test passes; covers all error codes in DDD §1 |
| 4 | API endpoint `POST /xxx` | P2 | 2h | 🟢 | 3 | No | Contract test passes; OpenAPI updated |
| 5 | UI form binding | P3 | 3h | 🟢 | 4 | Yes (with #6) | E2E happy path passes |
| 6 | Telemetry + logs | P4 | 1h | 🟢 | 3 | Yes (with #5) | Metrics visible in staging dashboard |

## Suggested implementation order
1 → 2 → 3 → 4 → (5 ∥ 6)

## Risk-first warning
Sub-task #1 is 🟠 — recommend doing it first and verifying rollback immediately after merging the migration.
```

## ⚠️ Hard rules / Quy tắc bắt buộc
- Sub-task without measurable DoD → reject, rewrite. / Sub-task không có DoD đo được → viết lại.
- Every 🟠/🔴 sub-task MUST include "rollback strategy" in its DoD. / Mọi sub-task 🟠/🔴 phải có rollback strategy trong DoD.
- If total estimate > 5 days → warn user "consider splitting into multiple PRs". / Nếu tổng >5 ngày → cảnh báo cân nhắc chia nhiều PR.

## ➡️ Next step / Bước tiếp theo
Move to **Stage 7: Implement** — run `/start-coding`. After each sub-task: stage → `/review-staged` → commit if READY.

**VI**: Chuyển sang Bước 7 — chạy `/start-coding`. Sau mỗi sub-task: stage → `/review-staged` → commit nếu READY.
