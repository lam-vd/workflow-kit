---
mode: agent
description: "Stage 7 of the Senior Workflow — Start coding a sub-task according to the FINAL spec and DDD plan. Use AFTER /breakdown-task when you are ready to implement the next sub-task. Reads the FINAL BD + DDD + sub-task list, identifies which sub-task to work on next (by priority or user request), loads relevant rules (clean-code, architecture) and skills (design-patterns), then generates production code + tests following the spec exactly. Stages changes with git add (no commit). After each sub-task completes, reminds user to run /review-staged to review and commit."
---

You are at **Stage 7: Start Coding** of the workflow.

> **VI**: Bước 7 — Bắt đầu code theo spec FINAL và DDD. Mỗi lần chạy = 1 sub-task.

## Preconditions / Điều kiện (verify before writing any code / xác nhận trước khi code)
1. ✅ Spec is `FINAL` — check `Status: FINAL` in both BD and DDD files. / Spec đã FINAL.
2. ✅ `/breakdown-task` (Stage 6) has been completed — sub-task list exists. / Đã chạy `/breakdown-task`.
3. ✅ If not → STOP, ask user to run the missing stage first. / Nếu chưa → yêu cầu chạy bước còn thiếu.

## Task / Nhiệm vụ

Implement the **next sub-task** (or the one user specifies) following the FINAL spec exactly.

**VI**: Code sub-task tiếp theo (hoặc sub-task user chỉ định), bám sát FINAL spec tuyệt đối.

## Steps

### 1. Load context (do this EVERY time)
Read these files in order:
```
1. docs/specs/<task>.md          → understand WHAT and WHY
2. docs/ddd/<task>.md            → understand HOW (API, schema, algorithms, diagrams)
3. Sub-task list from Stage 6    → identify current sub-task's DoD
4. .cursor/rules/clean-code.mdc  (or .github/instructions/clean-code.instructions.md)
5. .cursor/rules/architecture.mdc (or .github/instructions/architecture.instructions.md)
6. .agents/skills/design-patterns/SKILL.md (if patterns needed)
```

### 2. Identify current sub-task
- Check which sub-task is next (by priority P0 → P1 → P2 → P3 → P4).
- If user specifies a different one → use that.
- Print:
```
📌 Working on sub-task #N: <title>
   Priority: P1 | Est: 3h | Risk: 🟡
   DoD: <definition of done>
   Depends on: #<prev>
```

### 3. Plan before coding
Before writing any code, produce a brief **implementation plan** (5–15 lines):
- Which files to create / modify.
- Which layer each change belongs to (domain / application / infrastructure / UI).
- Key decisions and why (reference DDD section).
- Test approach for this sub-task.

Ask user: "Plan looks good? Proceed?" — wait for confirmation.

### 4. Implement
- Write production code following:
  - Architecture rules (layer boundaries, DI, pure domain).
  - Clean code rules (naming, size, no magic values).
  - DDD spec exactly (API contract, error codes, algorithms).
- Write tests alongside (TDD when possible):
  - Unit tests for business logic.
  - Integration tests if sub-task involves DB/external.
  - Cover happy path + edge cases listed in DDD.

### 5. Stage for review (do NOT commit here)
```sh
git add <files changed for this sub-task only>
```
- Print a **suggested commit message** (for `/review-staged` when READY):
  ```
  <type>(<scope>): <imperative summary>
  ```
- **DO NOT run `git commit`** — see `.cursor/rules/git-commit-policy.mdc`.

### 6. Verify DoD
After coding, check the sub-task's Definition of Done:
```markdown
## ✅ Sub-task #N DoD Check
- [x/] <DoD criterion 1>
- [x/] <DoD criterion 2>
- [x/] <DoD criterion 3>
```

If all pass → print:
```
✅ Sub-task #N complete. DoD met. Changes staged.
➡️ Next: run /review-staged — review staged diff, then commit if READY.
```

If any DoD item fails → continue fixing until met.

## Coding principles (from rules)

| Rule | Summary |
|---|---|
| Clean code | ≤20-line functions, meaningful names, no magic values, no dead code |
| Architecture | Layer direction: Controller → Application → Domain → Infra. Domain is pure. |
| Error handling | Catch at boundaries, typed errors with codes, log context not secrets |
| Security | Validate input at boundaries, authz on every endpoint, no hardcoded secrets |
| Tests | AAA structure, behavior-named, mock only at boundaries |

## ⚠️ Hard rules / Quy tắc bắt buộc

- **NEVER deviate from FINAL spec** — if you discover a needed change, STOP and ask: "Spec may need update. Return to Stage 3 or continue with assumption?" / KHÔNG lệch khỏi FINAL spec — nếu cần thay đổi, DẮNG và hỏi user.
- **NEVER skip tests** — every sub-task must have tests matching its DoD. / KHÔNG bỏ qua test — mọi sub-task phải có test khớp DoD.
- **NEVER ignore sub-task dependencies** — if sub-task #3 depends on #2, verify #2 is done first. / KHÔNG bỏ qua dependency giữa các sub-task.
- **NEVER run `git commit` in this stage** — stage only (`git add`); commit happens in `/review-staged` after READY. / KHÔNG commit ở bước này.
- **After completing a sub-task** → always remind: "Run `/review-staged` now." / Sau khi xong sub-task → nhắc chạy `/review-staged`.
- **NEVER touch unrelated code** — only modify files/functions directly required by the current sub-task's DoD. / KHÔNG đụng vào code không liên quan — chỉ sửa file/function trực tiếp cần cho DoD của sub-task hiện tại.

## 📤 Output structure

```markdown
## 📌 Sub-task #N: <title>

### Implementation plan
- Create `src/xxx/yyy.ts` (domain layer)
- Modify `src/api/routes.ts` (add endpoint)
- Create `tests/xxx/yyy.test.ts`
- Ref: DDD §4 (algorithm), DDD §1 (API contract)

### Code
<actual code changes>

### Tests
<actual test code>

### Suggested commit message (for /review-staged when READY)
```
feat(xxx): <imperative summary>
```

### DoD check
- [x] ...
- [x] ...

### ➡️ Next
Run `/review-staged` on staged changes. Commit only if verdict is READY.
```

## 🔁 Multiple sub-tasks in sequence

After `/review-staged` passes for sub-task #N (committed):
- User can run `/start-coding` again for sub-task #N+1.
- Repeat until all sub-tasks from Stage 6 are complete.
- Then → `/recheck-release` → `/create-pr`.
