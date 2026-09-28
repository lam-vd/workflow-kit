---
name: pr-review-comments
description: >-
  Paste-ready GitHub PR review comments for peer/Stage-8 reviews. Use when the
  user asks to review a PR/task and wants comments they can post (must / should /
  suggestion / nit), including repro steps and concrete fix hints. Complements
  code-review (full report) — this skill is the comment-output format only.
---

# Skill: PR Review Comments (paste-ready)

## When to use

- User asks to **review a PR / task** and wants comments to paste on GitHub.
- User asks to **tổng hợp comments**, “format must/should/suggestion/nit”, or similar.
- After `/review-staged` / `code-review` findings — convert report → paste-ready comments.

**Not** a replacement for `code-review` Full report (Intent & Coverage, verdict, boundary sweep). Emit **both** when Stage 8 needs the full report; emit **this format alone** when the user only wants PR comments.

## Mindset

- Critic, not approval-stamp.
- One finding = one comment block.
- Concrete file + lines + fix direction.
- English by default (team PR comments); Vietnamese only if user asks.
- Distinguish **must** vs **should** vs **suggestion** vs **nit** — never inflate nits to must.

## Severity → label → emoji

| Label | Emoji | Severity | Merge impact |
|-------|-------|----------|--------------|
| `must` | 🔴 | P0 / P1 | Block merge until fixed |
| `should` | 🟠 | P1-lite / strong P2 | Fix in same PR if cheap; otherwise track |
| `suggestion` | 🟡 | P2 | Recommended; can follow-up |
| `nit` | 🔵 | P3 | Optional / FYI / “skip — no change needed” |

ai-housemaker: request-spec UI asserts, tenant leaks, missing authz, broken Turbo contracts → **must** (🔴). Prefer `should` only when behavior is wrong but workaround exists.

Cutover / contract findings (any stack — see `feature-cutover-orphan-review`):
- Mutating transport auth weaker than HTTP for same resource → **must**
- Client stop/polling before required server phase (empty/wrong UI) → **must** / **should**
- Public route with zero product callers → **should**
- Ops-only enqueue (rake/CLI verified) → **nit** / resolve — not **must**
- Schema allow-list drift / unused locale·Stimulus target → **suggestion** / **nit**
- Missing prod backfill on stub/pre-deploy DoD → do **not** inflate to **must**

Lean facade findings (any stack — see `lean-facade-review`):
- `I18n.t` key missing in a required locale → **should** / **must** if EN/JA UX breaks
- `I18n.t(..., default: "…")` when key **already exists** → **nit** (drop `default:`) — not “add i18n”
- View-facing JSON passthrough getters → **skip** or optional store-accessor **nit**; do not demand raw hash in ERB
- Thin `col == CONST` predicates duplicated by helper UI map → **nit**
- Predicate with zero `app/` callers → **nit** Soft-dead

## Output template (verbatim structure)

Emit a numbered list. Each item:

```markdown
🔴 1. Short Title (Area)
📍 File: path/to/file.rb (lines X–Y)

must: One-line rule or problem statement.

Optional code fence showing bad snippet and/or ← remove / ← add hints.

**Repro:** (required for must/should when UI/functional)
1. Step…
2. Step…
3. Observe: …

**Suggestion:** Concrete fix (files, conditions, reuse existing pattern with 📍 if useful).

Priority: P0/P1 — blocks merge.   # or P2 / P3 as appropriate
```

Rules for each field:

| Field | Required? | Notes |
|-------|-----------|-------|
| Emoji + number + title | Yes | Title ≤ ~80 chars; include area tag in parentheses when helpful |
| `📍 File:` | Yes | Real path + line range when known; multiple `📍` ok |
| `must:` / `should:` / `suggestion:` / `nit:` | Yes | Exactly one label prefix matching the emoji |
| Code fence | When it clarifies | Prefer minimal diff-shaped hints (`# ← remove`) |
| `**Repro:**` | must + should (functional/UI) | Omit for pure lint/spec-style must if N/A — write `**Repro:** N/A (code/CI)` |
| `**Suggestion:**` | Yes | Actionable; cite reuse path with `📍` when pattern already exists |
| `Priority:` | Optional | Useful when ordering multiple musts |

### Skip / no-change nits

When declining a reviewer idea:

```markdown
🔵 N. Topic — Skip
📍 File: path (lines)

nit: Why current approach is fine — no change needed.
```

## Ordering

1. All 🔴 `must` (severity / user-impact first)
2. 🟠 `should`
3. 🟡 `suggestion`
4. 🔵 `nit`

End with a one-line verdict for the commenter:

```markdown
**PR verdict:** BLOCKED until must (#1–#N) are fixed. | READY (no must).
```

## Mapping from `code-review` findings

| code-review severity | This skill label |
|----------------------|------------------|
| 🔴 P0 | `must` |
| 🟠 P1 | `must` (or `should` if explicitly deferred by Product with Decision Log) |
| 🟡 P2 | `should` if user-visible / data risk; else `suggestion` |
| 🟢 P3 | `nit` |

## Relationship to other skills

| Skill | Role |
|-------|------|
| `code-review` | Full Stage-8 report (findings structure, coverage, verdict) |
| `rails-tl-review` | TL Summary (Critical / Suggestions / Praise) |
| `pr-review-comments` | **Paste-ready GitHub comments** (this file) |
| `branch-peer-review` | Peer/other-branch protocol + Task lock; uses this comment format |
| `ai-housemaker-review-checklist` | ahm P0/P1 gates (HARD BAN, Turbo, etc.) |

When profile **ai-housemaker** is active: HARD BAN UI-in-request-spec findings are always 🔴 `must`.

## Anti-patterns

- Vague “please fix this” without file/lines/suggestion.
- `must` for style preferences.
- Missing **Repro** on functional/UI must/should.
- Mixing multiple unrelated bugs in one numbered item.
- Vietnamese in PR comments unless the user asked for VI.
