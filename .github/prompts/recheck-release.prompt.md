---
mode: agent
description: "Stage 9a of the Senior Workflow — Release readiness gate before /create-pr. Verifies 6 categories: code quality (no TODO/FIXME/console.log/dead code), tests (unit + integration + CI green, coverage not regressed), spec compliance (all sub-tasks DoD, no out-of-scope), operational safety (rollback tested, feature flag for 🟠/🔴, telemetry added), documentation (README/CHANGELOG/OpenAPI updated), security (no secrets, input validation, authz). Outputs READY ✅ (all pass) or BLOCKED ❌ with specific failed gates and required actions."
---

You are at **Stage 9a: Release Readiness Check**.

> **VI**: Bước 9a — Kiểm tra sẵn sàng release. Nếu bất kỳ mục nào FAIL → kết quả là BLOCKED.

## Task / Nhiệm vụ
Verify each item in the gate checklist. If ANY item fails → result is `BLOCKED ❌`.

**VI**: Kiểm tra từng mục trong checklist. Bất kỳ mục nào fail → BLOCKED.

## Gate checklist / Danh sách kiểm tra

### A. Code quality / Chất lượng code
- [ ] `/review-staged` ran and has no remaining 🟠 or 🔴. / Đã chạy và không còn 🟠/🔴.
- [ ] Any unresolved 🟡 Medium findings are triaged and tracked (ticket + owner + target). / Mọi 🟡 chưa sửa phải được triage và có tracking.
- [ ] No `TODO` / `FIXME` / `XXX` in staged code. / Không có TODO/FIXME/XXX.
- [ ] No `console.log` / `print` / debugger statements. / Không có console.log/print/debugger.
- [ ] No commented-out code. / Không có code bị comment.

### B. Tests / Kiểm thử
- [ ] Unit tests pass locally: `<project-specific command>`. / Unit test pass local.
- [ ] Integration tests pass. / Integration test pass.
- [ ] CI green on the current branch. / CI xanh trên branch hiện tại.
- [ ] Coverage not regressed vs base branch. / Coverage không giảm so với base branch.

### C. Spec compliance / Tuân thủ spec
- [ ] All sub-tasks from `/breakdown-task` reach DoD. / Tất cả sub-task đạt DoD.
- [ ] All changes are within FINAL spec scope. / Mọi thay đổi nằm trong scope FINAL spec.
- [ ] No out-of-scope changes; if any → spec updated or PR split. / Không có thay đổi ngoài scope; nếu có → update spec hoặc tách PR.

### D. Operational safety / An toàn vận hành
- [ ] Migration has a tested rollback plan. / Migration có rollback plan đã test.
- [ ] Feature flag configured (if impact 🟠 / 🔴). / Feature flag (nếu 🟠/🔴).
- [ ] Telemetry / logs / metrics added for new flows. / Đã thêm telemetry/logs/metrics.
- [ ] User-facing error messages reviewed. / Đã review message lỗi user-facing.

### E. Documentation / Tài liệu
- [ ] README / CHANGELOG updated for user-facing changes. / README/CHANGELOG đã update.
- [ ] API docs (OpenAPI / Swagger) updated if API changed. / API docs update nếu đổi API.
- [ ] Spec changelog updated if spec changed after FINAL. / Spec changelog update nếu spec thay đổi sau FINAL.

### F. Security / Bảo mật
- [ ] No secrets / keys / tokens in code or git history. / Không có secret/key/token trong code.
- [ ] Input validation at system boundaries. / Validate input tại biên hệ thống.
- [ ] Authz check on every new endpoint. / Kiểm tra authz trên mọi endpoint mới.

## Steps to execute

1. Run `git diff --cached --name-only` to list files.
2. Grep for `TODO|FIXME|XXX|console\.log|print\(|debugger`:
   ```sh
   git diff --cached -G "TODO|FIXME|XXX|console\.log|debugger"
   ```
3. Read FINAL spec to verify coverage.
4. Read most recent `/review-staged` report.
5. Walk through each checklist item.

## 📤 Output

### When PASS
```markdown
# ✅ Release Readiness: READY

All gates passed:
- ✅ Code quality
- ✅ Tests
- ✅ Spec compliance
- ✅ Operational safety
- ✅ Documentation
- ✅ Security

➡️ Next: /create-pr
```

### When FAIL
```markdown
# ❌ Release Readiness: BLOCKED

## Failed gates
- ❌ **Tests** — Integration test `xxx` failing on CI.
- ❌ **Documentation** — README not updated for new endpoint `POST /xxx`.

## Required actions
1. Fix failing test (priority).
2. Update README §API.
3. Re-run `/recheck-release`.
```

## ⚠️ Hard rules
- DO NOT output `READY` if ANY item is uncertain — must verify concretely.
- DO NOT skip the "Security" gate even if the task seems unrelated.

## Medium handling gate
- `READY` is allowed with unresolved `🟡 Medium` only when all are tracked with ticket + owner + target date/milestone.
- Any unresolved `🟡` touching auth/authz, tenant scoping, payment, migration safety, or data integrity => treat as `🟠` and return `BLOCKED`.
