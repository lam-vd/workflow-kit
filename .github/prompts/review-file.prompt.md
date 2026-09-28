---
mode: agent
description: >-
  Review specific file(s) the user names, on your own branch, outside the Stage-8
  commit gate. Mandatory scope expansion (call sites) + base-branch comparison.
  Use /review-file. Report only — never print a commit command.
---

You are reviewing **specific file(s) the user named**, not a staged diff and not someone else's branch.

> **VI**: Review file lẻ do user chỉ định. **Bắt buộc** nới phạm vi ra call site và so với base branch — không kết luận từ mỗi nội dung file. Không phải commit gate → **không** in lệnh commit, **không** `git commit` / `git push`.

## Do not use this prompt when

- User is self-reviewing **staged** work before committing → `/review-staged`.
- User is reviewing **another developer's** branch/PR → `/review-branch`.
- User asks to *change* the file rather than assess it → `/start-coding`.

## Modes

| Mode | Trigger | Reads |
|------|---------|-------|
| `quick` | user says "review nhanh" / "quick" | `code-review` only; state explicitly this is **not** a full Stage-8 pass |
| `full` | **default** | `code-review` + `edge-case-boundary-review` + conditional skills (step 5) |

## Steps

1. **Read `.agents/skills/code-review/SKILL.md`** — methodology + report template. Mandatory in both modes. Do not improvise a template from memory.
2. **Scope expansion (mandatory).** A file cannot be judged alone.
   ```sh
   rg -n "<ClassName>|<module>|<public_method>" app/ lib/ spec/ config/
   ```
   Read every consumer found. Zero consumers ⇒ say so explicitly — that is itself a finding (dead code, or a symbol wired in a way grep missed).
3. **Compare against base — do not only read the current file.**
   ```sh
   BASE=$(git merge-base HEAD origin/develop 2>/dev/null \
       || git merge-base HEAD origin/main 2>/dev/null \
       || git merge-base HEAD main)
   git log --oneline "$BASE"..HEAD -- <path>
   git show "$BASE":<path>          # pre-change version; absent = new file
   git diff "$BASE"...HEAD -- <path>
   ```
   Map **every** branch / early return / guard in the old version onto the new one. Behavior that silently disappeared is the highest-value finding this prompt exists to catch.
   If the file is identical to base, say so — the review is then pure design review, not change review.
4. **Ground every claim in repo facts before writing it.** Assertions about behavior, security or limits must cite something read in this session, not general framework knowledge. Check the layer that actually decides:

   | Claim about | Verify in |
   |---|---|
   | nullability, uniqueness, defaults | `db/schema.rb` |
   | session, cookies, auth adapter | `config/initializers/*`, `config/environments/*` |
   | a limit / max / range | DB + model + controller + JS + locale — all five |
   | missing i18n | grep `config/locales/**` first |
   | dead code | grep before asserting; ops/rake/CLI callers count |
5. **`full` only — edge case & boundary sweep (mandatory unless the file is copy / CSS / locale-text only; when skipped, say so explicitly).**
   Read `.agents/skills/edge-case-boundary-review/SKILL.md`.
   - Build the **Boundary inventory**: every limit / range / comparison / date / collection / counter the file touches, each with **inside (`B-1`)**, **at (`B`)**, **outside (`B+1`)** behavior filled in.
   - Check **enforcement consistency** of each limit across DB / model / controller / JS / locale — a limit present in one layer but absent in another is the finding.
   - Sweep the `EDGE-*` catalog: `PASS` needs evidence, `N/A` needs a reason, blank = incomplete.
   - Pagination edges → cite `PAGE-*` from `paginated-list-patterns.mdc`; do not restate the rule.
   - Emit **§1d Edge case & boundary sweep** in the report.
   - A file with genuinely no numeric / temporal / collection boundary (pure dispatch, config, presenter) → still emit §1d, state that, and list the **type / nil / state** branches instead.
6. **`full` only — other conditional skills**, load only those the file actually matches:
   `dry-duplication-scan` reverse pass (new symbol) ·
   `functional-verification-review` (behavior change) ·
   `lean-facade-review` (thin getters / `col == CONST` predicates / `I18n.t` `default:`) ·
   `integration-regression-review` (UI / Hotwire / multi-step flow) ·
   `feature-cutover-orphan-review` (CRUD UI removed, domain kept) ·
   `structural-change-analysis` (split / move / delete).
   ai-housemaker → profile §8 + `rails-tl-review`. Doogo → **never** load ai-housemaker skills.
7. **Emit report** per `code-review` template. Every finding carries: severity · tag · whether it is **introduced by this change or pre-existing** · concrete fix hint.
8. **Close with scope honesty** — one line naming what you did *not* cover (tests not run, mode was `quick`, skill skipped and why).

## Severity calibration

| | |
|---|---|
| 🔴 / 🟠 | Introduced by this file **and** reachable in production |
| 🟡 | Pre-existing, or reachable only via impossible data — label `pre-existing`, never block on it |
| 🟢 | Style, naming, nit — recommend only, never demand |

Pre-existing behavior that this file merely moved is **not** a blocker. Say "unchanged from base" and move on.

## Hard rules

- **Never conclude from the named file alone** — steps 2 and 3 are not optional in `full` mode.
- **Verify infrastructure before raising a security finding.** Session fixation without reading `session_store.rb`, N+1 without reading the query, race without reading the index — all are speculation. Retract immediately and plainly if evidence contradicts you.
- Distinguish **introduced** vs **pre-existing** on every finding; conflating them turns a clean refactor into a false blocker.
- Boundary rows: **"outside" must name a concrete behavior** — `422` / clamp / no-op / redirect / raise. "Should be validated" is a 🟠 finding, not an answer.
- A limit enforced **only** in JS or HTML (`maxlength`, `disabled`, client validation) with no server-side rule → 🔴. Re-check by imagining the attribute stripped in DevTools.
- Do **not** auto-fix. Report; the user decides. If they then ask for a fix, keep it surgical (`karpathy-guidelines` §3.1) and re-state behavior parity branch by branch.
- Shared surface → call-site or opt-in (`shared-abstraction-safety`); never change a shared default for one caller.
- Do not inflate nits to blockers. Do not say "LGTM" without detail.
- **Never** print a commit command — this prompt is not a commit gate. **Never** `git commit` / `git push`.
- Report in the user's chat language; code, identifiers and commit text stay English.
