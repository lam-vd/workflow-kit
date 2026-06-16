# 🧭 Senior Engineer Workflow

> **EN**: Standard personal workflow — from task intake to merged PR. Applies to every feature / bugfix / refactor.
> **VI**: Workflow chuẩn cá nhân — từ lúc nhận task đến khi merge PR. Áp dụng cho mọi feature / bugfix / refactor.

## 🎯 Goals / Mục tiêu

- Reduce **input ambiguity** → reduce final-stage rework. / Giảm mơ hồ đầu vào → giảm rework cuối.
- Force **think before code**. / Buộc suy nghĩ trước, code sau.
- Document decisions for future onboarding / handover. / Tài liệu hoá quyết định để onboard / handover sau này.

---

## 📐 Overall Flow

```mermaid
flowchart TD
    A[📥 Task received] --> B[1. /analyze-task<br/>Initial analysis]
    B --> C[2. /grooming<br/>5W + Risks + Dependencies]
    C --> D{Clear enough?}
    D -->|No| E[❓ Ask back PO]
    E --> C
    D -->|Yes| F[3. /write-spec<br/>BD + DDD<br/>VI/EN/JP + flow diagrams]
    F --> G[4. /recheck-spec<br/>Audit + Score 0–10]
    G --> H{Score ≥ 8<br/>and no 🔴?}
    H -->|No, has 🔴| F
    H -->|No, < 8| H2{User decides}
    H2 -->|Fix| F
    H2 -->|Bypass| I[5. /check-spec<br/>FINAL LOCK<br/>+ Decision Log]
    H -->|Yes| I
    I --> J[6. /breakdown-task<br/>Sub-tasks]
    J --> K[7. Implement<br/>Code per skills/rules]
    K --> L[8. /review-staged<br/>Self-review]
    L --> M{Has 🟠/🔴?}
    M -->|Yes| K
    M -->|No| N[9. /recheck-release<br/>+ /create-pr<br/>VI/EN/JP]
    N --> O[✅ PR ready]
```

---

## 1. `/analyze-task` — Initial analysis / Phân tích sơ bộ

**EN purpose**: Decompose the request before committing.
**VI**: Bóc tách yêu cầu trước khi cam kết.

**Output checklist:**
- [ ] **Summary** (1–2 sentences in your own words / tóm tắt bằng từ của mình)
- [ ] **Why** — real business goal / mục tiêu kinh doanh thật
- [ ] **Scope IN / OUT** (OUT is critical to prevent scope creep)
- [ ] **≥2 approach options** with pros/cons
- [ ] **Impact level**: 🟢 / 🟡 / 🟠 / 🔴
- [ ] **Estimate (Suemori)**: `optimistic | realistic | pessimistic`
- [ ] **Open Questions** for PO

---

## 2. `/grooming` — 5 Whys + Risk + Dependencies

**EN purpose**: Find hidden issues BEFORE writing the spec.
**VI**: Tìm vấn đề ẩn TRƯỚC khi viết spec.

**Required output:**
- 5 Whys → root cause / root goal
- Risk Matrix (Tech / Business / Ops × likelihood × impact × mitigation)
- Dependencies (blocked-by / blocks / external)
- Edge cases (≥5)
- Non-functional requirements (perf / security / scalability / observability / i18n / a11y / compliance)
- Ask-back questions

---

## 3. `/write-spec` — Basic Design + Detail Design

**EN purpose**: Document decisions before coding.
**VI**: Tài liệu hoá quyết định trước khi code.

**Output**: 2 files (each tri-lingual EN / VI / JP):
- `docs/specs/<YYYY-MM-DD>-<slug>.md` — Basic Design (BD)
- `docs/ddd/<YYYY-MM-DD>-<slug>.md` — Detail Design Document (DDD)

**Required diagrams (must include):**
- BD: 1 high-level **flowchart** showing processing flow / sơ đồ luồng xử lý high-level.
- DDD: 1 **flowchart** (logical processing) + 1 **sequence diagram** (happy path) + 1 **sequence diagram** (error path) + (if schema change) 1 **ERD**.

> **Must read**: `.agents/skills/writing-bd/SKILL.md`, `.agents/skills/writing-ddd/SKILL.md`.

---

## 4. `/recheck-spec` — Self-audit with **scoring + level**

**EN purpose**: Critic-mode audit with quantified score.
**VI**: Audit chế độ phản biện, có chấm điểm.

### Pass criteria
- **Score ≥ 8.0 / 10** → APPROVED → `/check-spec`.
- **Score < 8.0** → NOT YET. Recommend back to `/write-spec`.
- **User is final decision-maker** — may bypass < 8 (logged in Decision Log).
- **Absolute hard rule**: any 🔴 Critical → BLOCKED, no bypass.

### 10-item Rubric (each 0 / 0.5 / 1.0)

1. Open Questions resolved
2. Edge cases coverage (≥5)
3. Sequence + flow diagrams (happy + error)
4. API contract completeness (schema + examples + error codes)
5. Migration & rollback (up / down / backfill / steps)
6. Quantitative test plan
7. Observability plan (metrics + logs + traces + alerts)
8. Security considerations
9. Performance budget (p50 / p95 / p99 + throughput)
10. Consistency & clarity

> N/A items → justify in 1 line, pro-rate score × 10 / applied items.

### Severity ↔ Score
| Severity | Effect |
|---|---|
| 🔴 Critical | Item = 0.0 + auto-block, no bypass |
| 🟠 High | Item = 0.0 |
| 🟡 Medium | Item = 0.5 |
| 🟢 Low | No deduction |

---

## 5. `/check-spec` — Final Lock

**5 gates** (all required):
1. ✅ `/recheck-spec` ran with scorecard.
2. ✅ Score ≥ 8.0 OR explicit user bypass logged.
3. ✅ NO 🔴 Critical (always blocks, never bypassable).
4. ✅ Stakeholder sign-off (manual confirm).
5. ✅ Spec committed to current branch.

→ Flip `Status: DRAFT` → `Status: FINAL`. Add `Locked at: <ISO>`, `Audit score: X.X / 10`. Append changelog. If bypassed: list open risks + follow-up.

---

## 6. `/breakdown-task` — Split into sub-tasks

- Each sub-task ≤ 4h.
- Each has measurable Definition of Done.
- Priority order: P0 schema/contract → P1 core logic / high-risk → P2 API/integration → P3 UI/glue → P4 polish.
- Mark parallel-able sub-tasks.

---

## 7. Implement — Triển khai

Before typing line 1:
- [ ] Re-read FINAL spec.
- [ ] Read `.cursor/rules/clean-code.mdc` (or `.github/instructions/clean-code.instructions.md`).
- [ ] Read `.cursor/rules/architecture.mdc`.
- [ ] Read relevant `.agents/skills/`.

While coding:
- Small commits, Conventional Commits messages.
- Tests alongside code (TDD when possible).
- After each sub-task → run `/review-staged` locally.

---

## 8. `/review-staged` — Self-review staged changes

Scope: only staged files (`git diff --cached`).

**8 review dimensions:**
1. Spec compliance (matches DDD?)
2. Clean code & naming
3. Architecture / layer boundaries
4. Error handling & logging
5. Security (input validation, secrets, OWASP top 10)
6. Performance (N+1, big-O, allocation)
7. Test coverage (happy + edge + error paths)
8. Backward compatibility / migration

**4 severity levels:** 🔴 Critical (block) / 🟠 High (must-fix) / 🟡 Medium (should-fix) / 🟢 Low (nitpick).

---

## 9. `/recheck-release` + `/create-pr`

### 9a. `/recheck-release` — Release readiness
6 gate categories: code quality, tests, spec compliance, operational safety, documentation, security. Output `READY ✅` or `BLOCKED ❌` with reasons.

### 9b. `/create-pr` — Tri-lingual PR
- **PR Title**: English (Conventional Commits, ≤72 chars).
- **Squash commit body**: English.
- **PR Description**: 3 collapsible blocks `EN / VI / JP`, all with Why / What / Impact / How to test / Spec / Checklist / Rollback.

---

## 🔁 When to revert

| Situation | Revert to |
|---|---|
| New edge case mid-implement | Stage 3 |
| PO changes requirements | Stage 2 |
| Review finds 🔴 from design flaw | Stage 3 |
| `/recheck-release` BLOCKED | Depends |

---

## 📎 Slash command index

| Command | Stage | File |
|---|---|---|
| `/analyze-task` | 1 | `.github/prompts/analyze-task.prompt.md` |
| `/grooming` | 2 | `.github/prompts/grooming.prompt.md` |
| `/write-spec` | 3 | `.github/prompts/write-spec.prompt.md` |
| `/recheck-spec` | 4 | `.github/prompts/recheck-spec.prompt.md` |
| `/check-spec` | 5 | `.github/prompts/check-spec.prompt.md` |
| `/breakdown-task` | 6 | `.github/prompts/breakdown-task.prompt.md` |
| `/review-staged` | 8 | `.github/prompts/review-staged.prompt.md` |
| `/recheck-release` | 9 | `.github/prompts/recheck-release.prompt.md` |
| `/create-pr` | 9 | `.github/prompts/create-pr.prompt.md` |
