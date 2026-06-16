---
mode: agent
description: "Stage 8 of the Senior Workflow — Self-review of staged changes ONLY (git diff --cached). Reviews against 8 dimensions (spec compliance, clean code, architecture, error handling, security/OWASP, performance, tests, backward compat) with 4-level severity (🔴 Critical / 🟠 High / 🟡 Medium / 🟢 Low). Reads the FINAL spec before reviewing. Outputs a level-grouped report with file:line references and concrete suggested fixes. Does NOT auto-fix code — only reports. Verdict: READY (no 🔴/🟠) or BLOCKED."
---

You are at **Stage 8: Review Staged Changes**.

> **VI**: Bạn đang ở Bước 8 — Tự review code đã staged trước khi merge.

## Task / Nhiệm vụ

Review ONLY the **staged** files in git, against 8 dimensions, with 4 severity levels.

**VI**: Chỉ review các file đã staged (`git diff --cached`), đánh giá theo 8 chiều chất lượng, phân loại lỗi theo 4 mức độ nghiêm trọng.

## Steps

1. **List staged files**:
   ```sh
   git diff --cached --name-only
   ```
2. **Get full diff**:
   ```sh
   git diff --cached
   ```
3. **Read relevant FINAL spec** in `docs/specs/` and `docs/ddd/`.
4. **Read rules**:
   - `.cursor/rules/clean-code.mdc` (or `.github/instructions/clean-code.instructions.md`)
   - `.cursor/rules/architecture.mdc`
   - `.agents/skills/code-review/SKILL.md`
5. **Review against the 8 dimensions** below.
6. **Emit report** in the format below.

## 8 Review Dimensions / 8 Chiều đánh giá

| # | Dimension | Look for | VI — Tìm gì? |
|---|---|---|---|
| 1 | Spec compliance | Matches DDD? Any missed requirement? | Khớp DDD không? Thiếu yêu cầu nào? |
| 2 | Clean code | Naming, function size, single responsibility, dead code | Tên biến rõ ràng? Hàm ngắn? Dead code? |
| 3 | Architecture | Layer boundaries, dependency direction, coupling | Tầng đúng hướng? Coupling thấp? |
| 4 | Error handling | Try/catch placement, error propagation, user-facing messages | Xử lý lỗi đúng chỗ? Message user-facing OK? |
| 5 | Security | Input validation, secrets, OWASP top 10, authz checks | Validate input? Không hardcode secret? Authz? |
| 6 | Performance | N+1, big-O, allocations, blocking I/O | N+1 query? Big-O hợp lý? Blocking I/O? |
| 7 | Tests | Coverage for happy + edge + error paths; brittleness | Cover đủ happy + edge + error? Test giòn? |
| 8 | Backward compat | Schema changes, API breaking, deprecation | Schema change có migration? API breaking? |

## Severity levels / Mức độ nghiêm trọng

- 🔴 **Critical / Nghiêm trọng** — block release. VD: SQL injection, mất dữ liệu, phá contract.
- 🟠 **High / Cao** — phải sửa trước khi merge. VD: thiếu error handling, O(n²) trên hot path.
- 🟡 **Medium / Trung bình** — nên sửa, có thể follow-up. VD: tên biến không rõ, thiếu test edge-case phụ.
- 🟢 **Low / Thấp (nitpick)** — tùy chọn. VD: thiếu comment, formatting.

## 📤 Output template

```markdown
# 🔍 Review Report — <task name / commit>
**Files reviewed**: 7
**Spec ref**: docs/ddd/<file>.md
**Reviewed at**: <timestamp>

## 🔴 Critical (block release)
- [ ] `path/file.ts:42` — SQL query uses string concatenation, exposed to injection.
  - **Fix**: switch to parameterized query.

## 🟠 High (must-fix before merge)
- [ ] `path/file.ts:88` — Does not catch error from external API; user gets 500.
  - **Fix**: wrap try/catch, return 502 with code `UPSTREAM_FAILED`.

## 🟡 Medium (should-fix, can follow up)
- [ ] `path/file.ts:120` — `doStuff` is 80 lines, violates SRP.

## 🟢 Low / nitpick
- [ ] `path/file.ts:5` — Missing JSDoc for public function.

## ✅ Clean (no issues)
- src/utils/format.ts
- src/types/user.ts

## 🚦 Verdict
- [ ] READY for `/recheck-release`
- [x] BLOCKED — fix 🔴 and 🟠 first
```

## ⚠️ Hard rules
- DO NOT auto-fix code — only report.
- Every issue MUST have: `file:line`, brief description, concrete suggested fix.
- If no 🔴 or 🟠 → verdict `READY`. Otherwise → `BLOCKED`.
- If a staged file is outside the spec's scope → flag as 🟠 "out-of-scope change".
