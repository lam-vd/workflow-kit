---
mode: agent
description: "Stage 7 — Start coding a sub-task per FINAL spec. Stages with git add only (no commit). After sub-task: remind /review-staged; user commits in terminal when READY."
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

## Project profile

If **ai-housemaker** (auto-detect or `(ai-housemaker)` flag) → read `common/profiles/ai-housemaker.md` §7 (lint gates, autocomplete, UI layout via `rails-ui-layouts`, Figma SVG handoff via `figma-svg-html-structure` / `figma-erb-styling-audit`, pre-flight checklist).

## Steps

### 1. Load context (do this EVERY time)
```
1. docs/specs/<task>.md + docs/ddd/<task>.md
2. Sub-task list from Stage 6 → current DoD
3. .agents/skills/dry-duplication-scan/SKILL.md (mandatory before writing new code)
4. .agents/skills/design-patterns/SKILL.md (if patterns needed)
5. Project profile — if ai-housemaker → common/profiles/ai-housemaker.md §7
```
Code style / layering: `clean-code.mdc` and `architecture.mdc` apply via **globs** on files you edit — do not load manually unless linter fails.

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

**DRY gate** — the plan MUST carry a Reuse Scan row for every **new** symbol it introduces (file, class, method, partial, Stimulus controller, CSS block, locale key, constant/enum):

| New symbol | Layers searched | Search terms | Hits | Decision |
|------------|-----------------|--------------|------|----------|

Reuse the Stage 1 scan if the capability is unchanged; re-scan when the sub-task adds something Stage 1 did not name. Preference: **Reuse > Extend opt-in > Adapt > New**. Extending a symbol with ≥2 consumers → `.cursor/rules/shared-abstraction-safety.mdc` (default behavior unchanged).

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
  - For every limit / range / date / collection the sub-task touches, cover **inside, at, and outside** the boundary — see `.agents/skills/edge-case-boundary-review/SKILL.md`. Stage 8 will sweep these anyway; writing them now is cheaper.

### 5. Stage for review (do NOT commit here)
```sh
git add <files changed for this sub-task only>
```
- Print a **suggested commit message** (user runs after `/review-staged` READY):
  ```
  feat(<scope>): part <N>: <area> - <imperative summary>
  ```
  Example: `feat(property-ui): part 2: land - scope routes under properties`
- **DO NOT** run `git commit` or `git push` — see `.cursor/rules/git-commit-policy.mdc`.

### 6. Verify DoD
After coding, check the sub-task's Definition of Done:
```markdown
## ✅ Sub-task #N DoD Check
- [x/] <DoD criterion 1>
- [x/] <DoD criterion 2>
- [x/] <DoD criterion 3>
- [x/] **Lint clean** (project skill: RuboCop + ERBLint on touched `.rb` / `.html.erb`)
```

If project has `static-analysis-lint` skill → run scoped lint before staging (see skill for commands).

If all pass → print:
```
✅ Sub-task #N complete. DoD met. Changes staged.
➡️ Next: run `/review-staged` — if READY, copy commit command and run in **your terminal**.
```

If any DoD item fails → continue fixing until met.

## Coding principles

Follow `karpathy-guidelines.mdc` + rules on edited files (`clean-code`, `architecture` via globs). See `docs/workflow/RULES-SKILLS-PROMPTS-MAP.md`.

## ⚠️ Hard rules / Quy tắc bắt buộc

- **NEVER deviate from FINAL spec** — if you discover a needed change, STOP and ask: "Spec may need update. Return to Stage 3 or continue with assumption?" / KHÔNG lệch khỏi FINAL spec — nếu cần thay đổi, DẮNG và hỏi user.
- **NEVER skip tests** — every sub-task must have tests matching its DoD. / KHÔNG bỏ qua test — mọi sub-task phải có test khớp DoD.
- **NEVER ignore sub-task dependencies** — if sub-task #3 depends on #2, verify #2 is done first. / KHÔNG bỏ qua dependency giữa các sub-task.
- **NEVER run `git commit` or `git push`** — stage only (`git add`); user commits after `/review-staged` READY. / KHÔNG commit/push — user commit ở terminal.
- **After completing a sub-task** → always remind: "Run `/review-staged` now." / Sau khi xong sub-task → nhắc chạy `/review-staged`.
- **NEVER touch unrelated code** — only modify files/functions directly required by the current sub-task's DoD. / KHÔNG đụng vào code không liên quan — chỉ sửa file/function trực tiếp cần cho DoD của sub-task hiện tại.
- **NEVER introduce a new symbol without a Reuse Scan row** — a search that failed to find an existing one is the justification. / KHÔNG tạo symbol mới nếu chưa có dòng Reuse Scan chứng minh đã tìm mà không có sẵn.

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

### Suggested commit message (user runs after /review-staged READY)
```
feat(property-ui): part <N>: <area> - <imperative summary>
```

### DoD check
- [x] ...
- [x] ...

### ➡️ Next
Run `/review-staged`. If READY → copy commit command into **your terminal** (preserves git author).
```

## 🔁 Multiple sub-tasks in sequence

After `/review-staged` READY for sub-task #N (user committed in terminal):
- User can run `/start-coding` again for sub-task #N+1.
- Repeat until all sub-tasks from Stage 6 are complete.
- Then → `/recheck-release` → `/create-pr`.
