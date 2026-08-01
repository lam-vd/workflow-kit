---
name: code-review
description: "Skill for performing self or peer code/PR review at Stage 8 of the Senior Workflow. Follows a structured methodology: (1) Findings grouped by severity with Hiện tượng/Likelihood/Ảnh hưởng/Cách tái hiện/Gợi ý per item; (2) Intent & Coverage mapping (requirements → code); (3) Specialist follow-up; (4) Positive observations; (5) Merge verdict. Critic mindset — review the diff, distinguish must-fix from nice-to-have, propose concrete fixes. 8 dimensions + 4 Lean Code Gates. 4-level severity (🔴 P0 Critical / 🟠 P1 High / 🟡 P2 Medium / 🟢 P3 Low). Flags out-of-scope changes. Anti-patterns to avoid as reviewer."
---

# Skill: Code Review / PR Review

## When to commit / Khi nào commit

- **`/review-staged` READY** — agent prints commit command; **user** runs `git commit` in terminal. See `.cursor/rules/git-commit-policy.mdc`.
- **Agent never** runs `git commit` or `git push`.
- Never commit during spec phases or inside `/start-coding` (stage there; user commits after review).

## Mindset / Tư duy

- **Critic hat on** — không phải approval-stamp.
- Review **the diff** (không đọc lại cả repo).
- Phân biệt rõ `must-fix` vs `nice-to-have`.
- **Propose concrete fixes** — không chỉ phàn nàn mà phải gợi ý cụ thể.
- **Lean & pragmatic** — ghét code rác, thừa, rườm rà.
- **Acknowledge good work** — ghi nhận điểm tốt, không chỉ tìm lỗi.

---

## Review Methodology / Phương pháp review

### Phase 1: Context gathering / Thu thập ngữ cảnh

1. Đọc PR description / commit message → hiểu intent.
2. Đọc FINAL spec (BD + DDD) nếu có.
3. List files changed, classify by layer (domain / application / infra / UI).
4. Chạy linters (rubocop, eslint, ruff, etc.) trên staged/changed files.

### Phase 2: Review against dimensions / Đánh giá theo chiều

Use the **8 dimensions table** below. Dimensions #2–#3 defer to rules (loaded via globs on staged code files). **Lean gates** → `karpathy-guidelines.mdc` §2–§3.

**Integration supplement** — only if staged paths include views/JS/locales/CSS/API clients:
- Read `.agents/skills/integration-regression-review/SKILL.md` — trace cross-layer flows, regression sweep.
- ai-housemaker: `.cursor/rules/hotwire-integration-patterns.mdc`
- Paginated index + search/filter: `.cursor/rules/paginated-list-patterns.mdc` (PAGE-*, PAGE-SEARCH-*)
- Multi-step / second JSON for same UI / early DOM commit: **Phase C3** in `integration-regression-review` (API-PAR-01, TURBO-TEMP-*)

**Deadcode / UI migration supplement** — when PR deletes views or claims “modal-only / old screen unused”:
- Read `.agents/skills/deadcode-ui-migration-review/SKILL.md` — call-site matrix + Dead-sure vs Fallback-only vs Shared-keep.

**Functional supplement** (controllers / services / routes / behavior change):
- Read `.agents/skills/functional-verification-review/SKILL.md` — run mapped tests, Manual QA script.
- Scope: `git diff --cached` **and** `git diff <base>...HEAD` for PR branch context.

**Edge case & boundary supplement** (mandatory unless the diff is copy/CSS/locale-text only):
- Read `.agents/skills/edge-case-boundary-review/SKILL.md` — Boundary inventory (inside `B-1` / at `B` / outside `B+1`), enforcement consistency across DB↔model↔controller↔JS, `EDGE-*` catalog sweep.
- Emit **§1d** in the report. Findings tagged `[edge-case]`.

**DRY reverse supplement** (diff introduces new classes / methods / partials / Stimulus controllers / CSS blocks / locale keys):
- Read `.agents/skills/dry-duplication-scan/SKILL.md` Phase 2 + Phase 4 — did the diff re-implement something that already exists? Cite `DUP-*`. Findings tagged `[dry]`.

Static diff review alone is **insufficient** for Turbo frame boundaries, URL/history state, and action-level regressions.

### Phase 3: Intent & Coverage / Đối chiếu yêu cầu ↔ code

Map từng yêu cầu trong spec/PR description → code tương ứng:
- **Khớp** — code implement đúng và đủ.
- **Một phần** — implement đúng hướng nhưng thiếu edge case / luồng phụ.
- **Chưa đủ** — luồng chính hoặc phụ bị bỏ sót.
- **Ngoài scope** — code không thuộc yêu cầu nào → flag.

### Phase 4: Specialist follow-up / Cần review chuyên sâu?

Xác định xem PR có cần thêm review từ chuyên gia không:
- Security audit (nếu có auth/payment/PII)?
- Performance audit (nếu có query phức tạp / high-traffic path)?
- UX review (nếu có thay đổi UI)?
- DBA review (nếu có migration)?

### Phase 5: Verdict / Kết luận merge

---

## 8 Review Dimensions / 8 Chiều đánh giá

| # | Dimension | Canonical detail |
|---|-----------|------------------|
| 1 | Spec compliance | FINAL DDD + sub-task DoD; flag out-of-scope |
| 2 | Clean code | `.cursor/rules/clean-code.mdc` |
| 3 | Architecture | `.cursor/rules/architecture.mdc` |
| 4 | Error handling & boundaries | Typed errors, no swallowed exceptions; boundary behavior → `edge-case-boundary-review` (`EDGE-*`) |
| 5 | Security | Authz, validation, secrets, OWASP — flag 🔴 on injection/IDOR |
| 6 | Performance | N+1, big-O, blocking I/O |
| 7 | Tests | Happy + boundary (inside/at/outside) + error; ai-housemaker → `ai-housemaker-rspec` (**HARD BAN** UI in `spec/requests/**`) |
| 8 | Backward compat | Migrations, API versioning |

**Lean gates (review pass):** YAGNI/bloat, dead code, DRY — see `karpathy-guidelines.mdc` §2–§3; duplication introduced by the diff → `dry-duplication-scan` (`DUP-*`). Spot-check tags: `[typo]` `[naming]` `[syntax]` `[security]` `[anti-pattern]` `[edge-case]` `[dry]`.

---

## 4-level Severity / 4 mức nghiêm trọng

| Level | Label | When / Khi nào | Ví dụ |
|---|---|---|---|
| 🔴 | P0 Critical | Block release — không merge được | SQL injection, mất dữ liệu, phá contract, luồng chính không chạy |
| 🟠 | P1 High | Must-fix trước merge | Thiếu error handling, O(n²) hot path, thiếu authz, dead code, out-of-scope bloat |
| 🟡 | P2 Medium | Should-fix, có thể follow-up | Tên biến chưa rõ, thiếu test edge-case phụ, DRY violation |
| 🟢 | P3 Low / nitpick | Tùy chọn | Comment thừa, formatting, naming preference |

---

## Finding structure / Cấu trúc mỗi phát hiện

Mỗi finding PHẢI có đủ 5 phần (theo mẫu PR review thực tế):

```markdown
### <Tóm tắt 1 dòng> — **<Severity label> (P<n>)**

- **Hiện tượng**: <Mô tả cụ thể vấn đề gì xảy ra, ở đâu trong code (file:line hoặc function name)>
- **Likelihood / Khả năng xảy ra**: **Cao / Trung bình / Thấp / Có điều kiện** — <giải thích kịch bản trigger>
- **Ảnh hưởng**: <Hậu quả nếu không fix — user thấy gì? Data sai thế nào? UX lệch ra sao?>
- **Điều kiện data** (P0/P1): <roles, record counts, seed state>
- **Điều kiện URL / state** (P0/P1 UI/Turbo): <path, query, modal open, frame id>
- **Cách tái hiện** (P0/P1 mandatory):
  1. Step 1
  2. Step 2
  3. Quan sát: <kết quả lỗi>
- **Verify nhanh** (optional): DevTools Network / `rspec path:line` / address bar
- **Gợi ý / Fix**: <Đề xuất cụ thể, actionable — code approach hoặc direction>
```

Tags: `[typo]` `[naming]` `[syntax]` `[security]` `[anti-pattern]` `[functional]` `[integration]` `[edge-case]` `[dry]`

> **VI**: Không bao giờ chỉ nói "có bug" mà không chỉ ra hiện tượng + cách fix. Không bao giờ chỉ nói "nên refactor" mà không giải thích impact.

---

## Full report template / Mẫu báo cáo Stage 8

Output report PHẢI theo cấu trúc sau (linters: `common/checklists/review-linters.md`):

```markdown
# 🔍 Code Review: <PR title hoặc commit summary>

| Key | Value |
|-----|-------|
| Date | <YYYY-MM-DD> |
| Branch | <branch name> |
| Base | <base branch> |
| Files reviewed | <N> |
| Spec ref | docs/ddd/<file>.md |

---

## TL Summary (ai-housemaker — when `rails-tl-review` ran)

### 🚨 Critical Issues
- … (or *None*) — must also appear as 🔴/🟠 Findings below

### 💡 Suggestions & Refactoring
- … architecture/perf only (or *None*) — typically 🟡

### ✅ Praise
- …

---

## 1b. Regression sweep (integration)

### Hotwire (if applicable)
| Pattern / check | Result | Evidence |
|-----------------|--------|----------|

### Paginated list + search (if applicable)
| Pattern / check | Result | Evidence |
|-----------------|--------|----------|

---

## 1c. Functional verification

| Command / check | Exit | Result |
|-----------------|------|--------|
| `<rspec/...>` | 0 / 1 | PASS / FAIL |

### Manual QA script
| # | Action | Preconditions | Steps | Expected | Result |
|---|--------|---------------|-------|----------|--------|

---

## 1d. Edge case & boundary sweep

### Boundary inventory
| # | Value | Limit | Inside (`B-1`) | At (`B`) | Outside (`B+1`) | Enforced at |
|---|-------|-------|----------------|----------|-----------------|-------------|

### Enforcement consistency
| Layer | Value | Match |
|-------|-------|-------|

### Catalog sweep (`EDGE-*`)
| ID | Case | Result | Evidence |
|----|------|--------|----------|

Deferred: `PAGE-*` (paginated-list-patterns), `CSS-OVERFLOW-01`.

---

## 1. Findings

### Spot-check sweep
| Category | Count | Worst severity |
|----------|-------|----------------|
| Typos | <n> | … |
| Naming | <n> | … |
| Syntax | <n> | … |
| Security | <n> | … |
| Anti-patterns | <n> | … |

### 1.1 <title> — P0 🔴 `[tag]`
- **Hiện tượng:** …
- **Likelihood:** …
- **Ảnh hưởng:** …
- **Cách tái hiện:** …
- **Gợi ý:** …

## 2. Intent & Coverage
| Yêu cầu | Đánh giá | Ghi chú |
|----------|----------|---------|

## 3. Specialist follow-up
- Security / Performance / UX / DBA / Manual QA scope

## 4. Positive observations (≥2)

## 5. Verdict
**READY ✅ / BLOCKED ❌** — P0/P1 list
```

Bugfix sub-tasks: add **Fix coverage** table per `integration-regression-review` Phase E (or `common/profiles/ai-housemaker.md` §8 bugfix).

---

## Spot-check categories (mandatory)

Scan staged diff for spot-check categories (see table above). Gom vào Spot-check sweep trong report template.

---

## Domain supplements / Checklist theo project

**ai-housemaker** (profile active): `common/profiles/ai-housemaker.md` §8 + `ai-housemaker-review-checklist` + **`rails-tl-review`** (TL Summary) — **do not** also load `ai-housemaker-review-patterns.mdc` body.

---

## Reviewer anti-patterns / Điều KHÔNG được làm

- "LGTM" không detail → không phải review.
- Comment theo preference cá nhân không có justification (`I prefer X`).
- Yêu cầu refactor ngoài scope PR → file separate issue.
- Block PR vì style preference, không phải bug.
- Chỉ tìm lỗi mà không ghi nhận điểm tốt.
- Finding không có "Gợi ý" cụ thể → không actionable → không hữu ích.
