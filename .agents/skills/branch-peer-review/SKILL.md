---
name: branch-peer-review
description: >-
  Peer-review another developer's branch (any task): reconstruct Task lock from
  spec+diff, then emit paste-ready GitHub comments (must/should/suggestion/nit).
  Use when reviewing someone else's PR/branch, reviewing on a different checkout,
  or when the user asks for a reusable branch-review prompt — not Stage-8
  /review-staged of the agent's own staged work.
---

# Branch peer review (other developer's branch)

**Human copy-paste prompt (canonical text):** [docs/prompts/branch-peer-review.md](../../../docs/prompts/branch-peer-review.md)  
**Comment format:** `pr-review-comments` (do not fork the template).  
**Own-work Stage 8:** `/review-staged` + `code-review` — this skill is **peer / other-branch** only.

## When to use

- User asks to review **another branch / another developer's PR / task**.
- User wants comments they can post, plus a **Task lock** so the reviewer does not reuse a previous feature's product rules.
- User pastes a diff, a branch name, or attaches files from a checkout that is not the author's.

**Not** a replacement for `/review-staged` (self-review + commit gate). Do not print a commit command.

## Mindset

- Critic, not approval-stamp.
- Chat **Vietnamese**; paste-ready comments **English** (unless user asks otherwise).
- Never invent issues. Never expand scope outside the diff. Never `git commit` / `git push`. Do not implement unless asked.
- If intent is ambiguous (≥1 material fork) → STOP with numbered questions.

## Load (diff-type only)

| Diff contains | Read |
|---------------|------|
| `spec/**` | Project RSpec skill/rule (`rspec-patterns` / `ai-housemaker-rspec` if ahm) |
| Shared helper/CSS/JS/Stimulus/partial | `shared-abstraction-safety` |
| Any code | `karpathy-guidelines` §3.1 |
| Comments output | `pr-review-comments` |
| ahm Rails/Hotwire/Pundit | `ai-housemaker-review-checklist` + `rails-tl-review` |
| New symbols | `dry-duplication-scan` reverse pass |
| Schema/fields | `field-impact-analysis` |
| Split/rename/delete | `structural-change-analysis` |
| Behavior boundaries | `edge-case-boundary-review` |
| CRUD UI removed / domain kept / new job·channel·FE template | `feature-cutover-orphan-review` |
| JSON passthrough getters / thin predicates / I18n `default:` | `lean-facade-review` |

## Hard rules

### Request spec ≠ UI (HARD BAN → 🔴 must in `spec/requests/**`)

Forbidden in `response.body`: JA/UI copy, `I18n.t(...)` as HTML, BEM/CSS scans, Stimulus `data-*` wiring, overlay ids used only for layout.

Gray zone OK: `*Controller::*FRAME_ID`, form param names, path helpers, `%(..."#{record.attr}"...)`.

If copy/CSS-only would fail the spec while HTTP/DB is unchanged → wrong layer.

### Shared surface

Call-site first. Shared API change must be **opt-in** (default unchanged). Grep consumers before requesting a shared edit.

### Surgical

Comments must map to **this task's DoD** or a ship risk in the diff. Do not flag pre-existing legacy unless the author introduced a regression.

### Cutover & contracts

When the diff removes a UI/CRUD surface or adds async/realtime:

- Grep before “unused” — **Ops-only** (rake/CLI/seed) ≠ Dead-sure.
- Unwired public routes / FE URL values never read → comment with evidence.
- Mutating transport auth must match HTTP policy for the same resource (`CONTRACT-AUTHZ-01`).
- Client stop/polling must not complete before a required server phase (`CONTRACT-ASYNC-01`).
- Do not escalate missing prod backfill on stub/pre-deploy DoD as 🔴.

Canonical: `feature-cutover-orphan-review`.

## Build Task lock first (never reuse another task's product)

From spec/DDD/commits/diff, 5–12 bullets **before** commenting:

- User-visible behavior that must not be “improved” away
- Authz / tenant / discarded-record policy
- Turbo: frame vs full page, 422 vs redirect
- Client-only vs server-trusted (ignore unowned query params)
- i18n: one key per distinct string; templates over twin “0-count” keys
- Load-bearing “dead-looking” wiring (e.g. overlay outside Stimulus host)
- Adjacent feature `data-*` / templates on the same element that are **out of scope**

If spec and code disagree, say so. Do not silently prefer code.

## Protocol

1. Scope diff: `git diff base...HEAD` (default base `main` / PR base). Ignore unrelated dirty files.
2. Write **Task lock**.
3. Per file: intent → layer → blast radius → authz/tenant → Turbo/HTTP → dead vs load-bearing → specs vs skills → drive-by.
4. Emit comments via `pr-review-comments` (must → should → suggestion → nit). Include **Skip** nits when the author already did the right non-obvious thing.
5. Coverage gaps at the **correct** layer — never recommend UI asserts in request specs.
6. **Lesson** (5–8 lines): 2–3 transferable review instincts this diff trains.

## Output shape

Follow [docs/prompts/branch-peer-review.md](../../../docs/prompts/branch-peer-review.md) sections **0–6**:

0. Task lock  
1. Diff map  
2. File-by-file (≤3 “what is correct” bullets + findings)  
3. Paste-ready GitHub comments  
4. Coverage gaps  
5. Skip / no-change  
6. Lesson  

**PR verdict:** one line (`BLOCKED` until musts / `READY`).

## Relationship

| Skill | Role |
|-------|------|
| `code-review` | Own-work Stage-8 **Full report** |
| `pr-review-comments` | Comment block template only |
| `branch-peer-review` | Peer/other-branch protocol + Task lock (this file) |
| `rails-tl-review` | ahm TL pass when profile active |
