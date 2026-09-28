---
mode: agent
description: "Stage 8 — Self-review staged changes + PR-branch context. Reads code-review skill; ai-housemaker adds rails-tl-review TL pass. Integration + functional verification when behavior/UI changes. On READY: print commit command for user (never run git commit or git push)."
---

You are at **Stage 8: Review Staged Changes**.

> **VI**: Tự review `git diff --cached` + ngữ cảnh branch PR. Chạy test khi đổi behavior. READY → in lệnh commit cho user chạy terminal — agent **không** `git commit` / `git push`.

**Peer / other-developer branch:** if the user wants to review **someone else’s PR/branch** (not own staged commit gate) → stop this flow. Use `/review-branch` + `.agents/skills/branch-peer-review/SKILL.md`. Do **not** print a commit command for their work.

## Project profile

If **ai-housemaker** (auto: cwd/paths contain `ai-housemaker/` or Rails+Hotwire repo; override: `(ai-housemaker)` or `/review-staged ai-housemaker`) → read `common/profiles/ai-housemaker.md` §8 (+ §8 bugfix when sub-task is fix/bug/regression).

If **telemedease** (auto: `telemedease/`; override: `(telemedease)` or `/review-staged telemedease`) → read `common/profiles/telemedease.md` §8 (idor/PayJP/portal; **no** `rails-tl-review` / Hotwire).

If **skal** (auto: `skal/` or `port_jp/skal`; override: `(skal)` or `/review-staged skal`) → read `common/profiles/skal.md` §8 (idor/ads CTR/Handsaw; **no** `rails-tl-review` / Hotwire / teleme-payment).

If **manet-adwords-ocv** (auto: `manet-adwords` / `manet-adwords-analysis--offline-cv`; override: `(ocv)` or `/review-staged ocv`) → read `common/profiles/manet-adwords-ocv.md` §8 (Sheets/API/cron/secrets; **no** Rails profiles).

If **manet** (auto: `manet/` / `port_jp/manet` but **not** `manet-adwords`; override: `(manet)` or `/review-staged manet`) → read `common/profiles/manet.md` §8 (idor/Super/Handsaw; FX untouched; **no** Hotwire / teleme-payment).

If **thira** (auto: `thira/` or `port_jp/thira`; override: `(thira)` or `/review-staged thira`) → read `common/profiles/thira.md` §8 (`kw_id` no-bleed / Cypress; **no** Rails).

If **evekatsu** (auto: `evekatsu/` or `port_jp/evekatsu`; override: `(evekatsu)` or `/review-staged evekatsu`) → read `common/profiles/evekatsu.md` §8 (routes/cache/PC-SP/idor; **no** Hotwire).

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
4a. **Edge case & boundary sweep (mandatory unless diff is copy/CSS/locale-text only):** read `.agents/skills/edge-case-boundary-review/SKILL.md`
    — build the **Boundary inventory**: every limit / range / comparison / date / collection / counter in the diff, with **inside (`B-1`), at (`B`), outside (`B+1`)** behavior filled in.
    — check **enforcement consistency** of each limit across DB / model / controller / JS / locale.
    — sweep the `EDGE-*` catalog; `PASS` needs evidence, `N/A` needs a reason, blank = incomplete.
    — pagination edges → cite `PAGE-*` from `paginated-list-patterns.mdc`, do not restate.
    — emit **§1d Edge case & boundary sweep** in the report.
4b. **ai-housemaker TL pass (when profile active):** read `.agents/skills/rails-tl-review/SKILL.md`
    — scan Architecture / Security+Tenant / Perf+DB / Hotwire+Stimulus / Testing against staged diff.
    — **Skip P3 nitpicks** that do not affect architecture, security, performance, or correctness.
    — Emit **TL Summary** (🚨 Critical / 💡 Suggestions / ✅ Praise) inside the Full report per that skill.
4c. Still emit Full report sections (Findings with severity, Intent & Coverage, Verdict). TL Critical ≡ 🔴/🟠 → BLOCKED.
5. **Run:** `common/checklists/review-linters.md` on staged files
6. **Conditional** (only if staged paths match — see `docs/workflow/RULES-SKILLS-PROMPTS-MAP.md`):
   - UI/integration → `integration-regression-review` skill (+ `hotwire-integration-patterns.mdc` if Hotwire)
   - UI migration / orphaned show→modal partials → `deadcode-ui-migration-review`
   - Partial ship / CRUD UI removed / domain kept / new job·channel·FE URL template → `feature-cutover-orphan-review`
   - JSON passthrough getters / thin `col == CONST` predicates / `I18n.t(..., default:)` → `lean-facade-review`
   - Overflow / long text in detail or modal → `css-safe-text-overflow` + CSS-OVERFLOW-01
   - Behavior change → `functional-verification-review` skill
   - New symbol added (class/method/partial/Stimulus/CSS block/locale key) → `dry-duplication-scan` **reverse pass**: does the diff re-implement something that already exists? Cite `DUP-*`
   - List/search/pagination → `paginated-list-patterns.mdc`
   - ai-housemaker → profile §8 + `ai-housemaker-review-checklist` + **`rails-tl-review`** (**not** `ai-housemaker-review-patterns.mdc` body)
7. **Bugfix sub-task** → `integration-regression-review` Phase E or profile §8 bugfix
7a. **Apply the evidence bar** — tag every finding; demote or drop what does not clear it (see below).
8. **Emit report** per code-review Full report template (+ TL Summary when 4b ran), preceded by **§0 Plain-language explanation**
9. **Commit gate** (READY only) — **print for user, do NOT execute**:
    ```bash
    git commit -m "$(cat <<'EOF'
    <type>(<scope>): part <N>: <area> - <imperative summary>
    EOF
    )"
    ```
    - Use multi-phase format when task has phases: `feat(property-ui): part 2: land - scope routes under properties`
    - Print: `✅ READY — run the commit above in **your terminal** (keeps your git author).`
    - **DO NOT** run `git commit` or `git push`.
    - See `.cursor/rules/git-commit-policy.mdc`.

## Evidence bar (findings must earn their place)

Tag every finding with one tier. The tier decides whether it may be reported, and whether it may block.

| Tier | Meaning | Allowed in Findings? |
|---|---|---|
| `[verified]` | You ran something this session that proves it — test, script, query, quoted command output | Yes — lead with these; only these may be stated as observed fact |
| `[deterministic]` | Repro follows from code with no timing/luck; not executed | Yes — carry the steps and say you did not run them |
| `[mechanism]` | Certain from code, but repro needs concurrency/load/environment you lack | Yes — say plainly it was not reproduced |
| `[inference]` | Rests on framework/gem behavior you did not read this session | **No** — *Unverified* list + the one command that settles it |
| `[hypothetical]` | Needs a precondition that does not exist (leaked secret, unreachable state) | **No** — at most one line in *Design questions* |

Since staged work is a commit gate: **only `[verified]` and `[deterministic]` findings may set BLOCKED.** A `[mechanism]` finding worth blocking on must first be promoted by actually reproducing it. Run the cheap check instead of guessing; if execution contradicts the claim, retract it plainly.

## §0 Plain-language explanation (mandatory, before the tables)

Written for someone who did not read the diff. Per reportable finding, in prose, no jargon, no bare `file:line`:

1. **How this part is supposed to work** — the real-world mechanic in one or two sentences.
2. **What actually happens** — the wrong behavior, same vocabulary.
3. **Who feels it and when** — user-visible symptom, or "nothing today, but…" said honestly.

Close with one sentence naming the shared root cause when findings have one. Severity tables, `EDGE-*` IDs and boundary inventories come **after** §0 — never make the reader decode them to learn what is wrong.

## Verdict

| Verdict | Conditions |
|---------|------------|
| **READY** | No 🔴/🟠; staged-related tests pass; regression/functional/boundary sections complete |
| **BLOCKED** | Any 🔴/🟠 that is `[verified]` or `[deterministic]`; related test fail; integration P1 FAIL; Manual QA FAIL without NOT RUN reason; any boundary row with **outside-behavior undefined** on a write path |

ai-housemaker extra gates: see profile §8 (Brakeman exit 0, Hotwire PAGE-* P1).

## Hard rules

- DO NOT auto-fix — report only (user fixes → re-stage → re-run).
- **NEVER** run `git commit` — user commits after READY (author identity).
- **NEVER** run `git push` — user pushes when they choose.
- Do not print commit command if verdict is BLOCKED (🔴/🟠 or failed staged-related tests).
- P0/P1 findings: Hiện tượng + Likelihood + Ảnh hưởng + Gợi ý (+ data/URL preconditions + Cách tái hiện for UI).
- MUST include **§0 Plain-language explanation** before any table; a report opening with a boundary table is incomplete.
- Every finding carries an **evidence tier**; `[inference]` / `[hypothetical]` never appear as numbered findings, and `[mechanism]` alone never sets BLOCKED.
- MUST include: Spot-check sweep, Intent & Coverage, Positive observations (≥2), Regression + Functional sections when applicable.
- MUST include **§1d Edge case & boundary sweep** unless the diff is copy/CSS/locale-text only — and say so explicitly when skipped.
- Boundary rows: "outside" must name a concrete behavior (422 / clamp / no-op / redirect). "Should be validated" is a 🟠 finding, not an answer.
- A limit enforced only in JS/HTML with no server rule → 🔴.
- ai-housemaker: MUST include **TL Summary** when `rails-tl-review` ran; Critical Issues ≡ 🔴/🟠.
- Tag findings: `[typo]` `[naming]` `[syntax]` `[security]` `[anti-pattern]` `[functional]` `[integration]` `[edge-case]` `[dry]`.
- Out-of-scope staged file → 🟠.
- DO NOT say "LGTM" without detail.
