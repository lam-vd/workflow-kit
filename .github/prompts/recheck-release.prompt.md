---
mode: agent
description: "Stage 9a of the Senior Workflow — Release readiness gate before /create-pr. Verifies 6 categories: code quality (no TODO/FIXME/console.log/dead code), tests (unit + integration + CI green, coverage not regressed), spec compliance (all sub-tasks DoD, no out-of-scope), operational safety (rollback tested, feature flag for 🟠/🔴, telemetry added), documentation (README/CHANGELOG/OpenAPI updated), security (no secrets, input validation, authz). Outputs READY ✅ (all pass) or BLOCKED ❌ with specific failed gates and required actions."
---

You are at **Stage 9a: Release Readiness Check**.

## Task
Verify each item in the gate checklist. If ANY item fails → result is `BLOCKED ❌`.

## Gate checklist

### A. Code quality
- [ ] `/review-staged` ran and has no remaining 🟠 or 🔴.
- [ ] No `TODO` / `FIXME` / `XXX` in staged code.
- [ ] No `console.log` / `print` / debugger statements.
- [ ] No commented-out code.

### B. Tests
- [ ] Unit tests pass locally: `<project-specific command>`.
- [ ] Integration tests pass.
- [ ] CI green on the current branch.
- [ ] Coverage not regressed vs base branch.

### C. Spec compliance
- [ ] All sub-tasks from `/breakdown-task` reach DoD.
- [ ] All changes are within FINAL spec scope.
- [ ] No out-of-scope changes; if any → spec updated or PR split.

### D. Operational safety
- [ ] Migration has a tested rollback plan.
- [ ] Feature flag configured (if impact 🟠 / 🔴).
- [ ] Telemetry / logs / metrics added for new flows.
- [ ] User-facing error messages reviewed.

### E. Documentation
- [ ] README / CHANGELOG updated for user-facing changes.
- [ ] API docs (OpenAPI / Swagger) updated if API changed.
- [ ] Spec changelog updated if spec changed after FINAL.

### F. Security
- [ ] No secrets / keys / tokens in code or git history.
- [ ] Input validation at system boundaries.
- [ ] Authz check on every new endpoint.

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
