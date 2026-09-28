---
mode: agent
description: >-
  Peer-review another developer's branch (not own Stage-8 staged work). Reconstruct
  Task lock from spec+diff; emit paste-ready GitHub comments. Use /review-branch.
  Do not print a commit command.
---

You are doing a **peer review of another developer’s branch** (not your own staged commit gate).

> **VI**: Review branch/PR của người khác. Tự viết **Task lock** từ spec+diff của *task này*. Comment GitHub English. **Không** `git commit` / `git push`. **Không** in lệnh commit.

## Do not use this prompt when

- User is self-reviewing **staged** work before they commit → `/review-staged` + `code-review`.

## Steps

1. Read `.agents/skills/branch-peer-review/SKILL.md`
2. Read `.agents/skills/pr-review-comments/SKILL.md` (comment template — do not fork)
3. Scope:
   ```sh
   BASE=$(git merge-base HEAD origin/main 2>/dev/null || git merge-base HEAD main)
   git branch --show-current
   git log --oneline "$BASE"..HEAD | head -20
   git diff --stat "$BASE"...HEAD
   git diff "$BASE"...HEAD
   ```
   Default base: PR base if known, else `main`. Ignore unrelated dirty files unless the user included them.
4. Load **diff-type** skills only (table in the skill). ai-housemaker → profile §8 + checklist + `rails-tl-review` when Rails/Hotwire. Partial ship / CRUD cutover / new job·channel → `feature-cutover-orphan-review`. Thin JSON getters / predicates / `I18n.t` `default:` → `lean-facade-review`.
5. Emit output sections **0–6** per `docs/prompts/branch-peer-review.md` (Task lock first).
6. Paste-ready comments via `pr-review-comments`. End with one-line **PR verdict**.

## Hard rules

- Reconstruct Task lock for **this** branch. Never reuse another feature’s product lock.
- HARD BAN: no UI asserts in `spec/requests/**` → 🔴 `must` on ai-housemaker.
- Shared surface: call-site / opt-in. Do not request shared default changes for one screen.
- Ops rake/CLI path ≠ “no app callers → must delete”. Cite `feature-cutover-orphan-review` classes.
- Grep locales before “missing i18n”; redundant `I18n.t` `default:` → nit (`lean-facade-review`).
- Do **not** auto-fix. Do **not** print a `git commit` command.
