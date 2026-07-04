---
mode: agent
description: "Stage 8 — Self-review staged changes + PR-branch context. Reads code-review skill; integration + functional verification when behavior/UI changes. ai-housemaker: auto-loads common/profiles/ai-housemaker.md. Commit only if READY."
---

You are at **Stage 8: Review Staged Changes**.

> **VI**: Tự review `git diff --cached` + ngữ cảnh branch PR. Chạy test khi đổi behavior. Commit chỉ khi READY.

## Project profile

If **ai-housemaker** (auto: cwd/paths contain `ai-housemaker/` or Rails+Hotwire repo; override: `(ai-housemaker)` or `/review-staged ai-housemaker`) → read `common/profiles/ai-housemaker.md` §8 (+ §8 bugfix when sub-task is fix/bug/regression).

## Steps

1. **PR branch context:**
   ```sh
   BASE=$(git merge-base HEAD origin/main 2>/dev/null || git merge-base HEAD main)
   git branch --show-current
   git log --oneline "$BASE"..HEAD | head -20
   git diff --stat "$BASE"...HEAD
   ```
2. **Staged:**
   ```sh
   git diff --cached --name-only
   git diff --cached
   ```
3. **Read FINAL spec** in `docs/specs/` and `docs/ddd/`.
4. **Read once (mandatory):** `.agents/skills/code-review/SKILL.md` — methodology + **report template**
5. **Run:** `common/checklists/review-linters.md` on staged files
6. **Conditional** (only if staged paths match — see `docs/workflow/RULES-SKILLS-PROMPTS-MAP.md`):
   - UI/integration → `integration-regression-review` skill (+ `hotwire-integration-patterns.mdc` if Hotwire)
   - Behavior change → `functional-verification-review` skill
   - List/search/pagination → `paginated-list-patterns.mdc`
   - ai-housemaker → profile §8 + `ai-housemaker-review-checklist` skill (**not** `ai-housemaker-review-patterns.mdc` body)
7. **Bugfix sub-task** → `integration-regression-review` Phase E or profile §8 bugfix
8. **Emit report** per code-review Full report template
9. **Commit gate** (READY only):
    ```sh
    git commit -m "$(cat <<'EOF'
    <type>(<scope>): <imperative summary>

    EOF
    )"
    ```
    Print: `✅ Committed sub-task #N — <hash short>`
    See `.cursor/rules/git-commit-policy.mdc`.

## Verdict

| Verdict | Conditions |
|---------|------------|
| **READY** | No 🔴/🟠; staged-related tests pass; regression/functional sections complete |
| **BLOCKED** | Any 🔴/🟠; related test fail; integration P1 FAIL; Manual QA FAIL without NOT RUN reason |

ai-housemaker extra gates: see profile §8 (Brakeman exit 0, Hotwire PAGE-* P1).

## Hard rules

- DO NOT auto-fix — report only (user fixes → re-stage → re-run).
- **ONLY this stage** may `git commit` in the implementation loop — and only when READY.
- NEVER commit with open 🔴/🟠 or failed staged-related tests.
- P0/P1 findings: Hiện tượng + Likelihood + Ảnh hưởng + Gợi ý (+ data/URL preconditions + Cách tái hiện for UI).
- MUST include: Spot-check sweep, Intent & Coverage, Positive observations (≥2), Regression + Functional sections when applicable.
- Tag findings: `[typo]` `[naming]` `[syntax]` `[security]` `[anti-pattern]` `[functional]` `[integration]`.
- Out-of-scope staged file → 🟠.
- DO NOT say "LGTM" without detail.
