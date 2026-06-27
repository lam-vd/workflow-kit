---
mode: agent
description: "Stage 5 of the Senior Workflow — FINAL spec lock. Gates entry to coding (Stage 7). Verifies 5 conditions: scorecard exists, score ≥ 8.0 OR explicit user bypass, NO 🔴 Critical (never bypassable), stakeholder sign-off, spec files saved on disk. On pass: flips Status DRAFT → FINAL, adds Locked-at timestamp + Audit-score, appends changelog. Does NOT git commit — see git-commit-policy. If bypass: changelog must list open risks + follow-up plan. Refuses to lock when 🔴 exists regardless of user request."
---

You are at **Stage 5: Final Spec Lock**.

> **VI**: Bước 5 — Khoá spec thành FINAL. Chặn lối vào giai đoạn code nếu chưa đạt điều kiện.

## Preconditions / Điều kiện (verify ALL 5 / xác nhận đủ 5)
1. ✅ `/recheck-spec` ran with a clear scorecard. / Đã chạy `/recheck-spec` có scorecard rõ ràng.
2. ✅ **Score ≥ 8.0 / 10** OR user has explicitly approved a bypass. / Điểm ≥8.0 hoặc user đã chấp nhận bypass.
3. ✅ **NO 🔴 Critical issue** (Critical always blocks, regardless of user decision). / KHÔNG có lỗi 🔴 (🔴 luôn chặn, không bypass được).
4. ✅ Stakeholder / PO sign-off (manual user confirmation). / PO đã sign-off.
5. ✅ Spec files are saved on disk (BD + DDD). Uncommitted working-tree changes are OK. / File spec đã lưu trên disk; **không** cần `git commit`.

If ANY condition fails → STOP, print the reason, ask user to fix.

### 🚦 Score gate logic

| Score | 🔴 Critical | Action |
|---|---|---|
| ≥ 8.0 | 0 | ✅ Proceed to lock FINAL |
| ≥ 8.0 | ≥1 | ❌ BLOCKED — fix 🔴 first, no bypass |
| < 8.0 | 0 | ⚠️ Ask user: "Score X.X < 8. Bypass or revise?" — proceed only if user chooses bypass |
| < 8.0 | ≥1 | ❌ BLOCKED |

## Task

1. Open the BD file `docs/specs/*.md` and DDD file `docs/ddd/*.md` for the task.
2. Change `Status: DRAFT` → `Status: FINAL`.
3. Add line `Locked at: <ISO timestamp>` immediately after `Status`.
4. Add line `Audit score: X.X / 10` immediately after `Locked at`.
5. Append the following changelog block at the bottom of EACH file:

```markdown
---
## Changelog
- <YYYY-MM-DD> — DRAFT → FINAL by <author>
  - Audit score: X.X / 10 via /recheck-spec
  - Stakeholders signed off: <names>
  - [If bypassed] ⚠️ Bypassed at score X.X with user approval. Open risks: <list>. Follow-up: <task/PR>.
```

6. Ask the user: "Have you verified all 5 preconditions? (score ≥ 8 or intentional bypass, no 🔴, sign-off received)" — wait for `yes` before saving FINAL metadata to the files.
7. **DO NOT run `git commit`** — see `.cursor/rules/git-commit-policy.mdc`. After saving files, print:

```
✅ Spec FINAL locked.
- BD: docs/specs/...
- DDD: docs/ddd/...
- Audit score: X.X / 10

➡️ Next: /breakdown-task
```

## ⚠️ Hard rules / Quy tắc bắt buộc
- **NEVER run `git commit`** in this stage — no `docs: lock spec FINAL` commits. / KHÔNG commit ở bước này.
- NEVER auto-flip to `FINAL` without user confirmation. / KHÔNG tự động chuyển FINAL mà không user xác nhận.
- NEVER lock FINAL while 🔴 Critical exists (even if user requests). / KHÔNG lock khi còn 🔴 dù user yêu cầu.
- When user bypasses below 8 — Decision Log + Changelog MUST list open risks + follow-up plan. / Khi bypass dưới 8 — phải ghi risk + follow-up plan.
- DO NOT modify any other content beyond the 4 changes above (status, locked-at, score, changelog). / KHÔNG sửa gì khác ngoài 4 thay đổi trên.
- After FINAL — any spec change must create a new file (`docs/specs/<date>-<slug>-v2.md`) or bump version. / Sau FINAL, mọi thay đổi spec phải tạo file mới hoặc bump version.
