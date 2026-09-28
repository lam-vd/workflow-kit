---
name: lean-facade-review
description: >-
  Stage 7–8 / peer-review skill for thin model facades: JSON/hash passthrough
  getters, string-compare predicates that only mirror a column, I18n.t calls that
  duplicate locale keys via hard-coded default:, and UI branching split between
  model predicates and helpers. Stack-agnostic. Prefer keep view-friendly APIs;
  do not inflate nits to must. Complements dry-duplication-scan and feature-cutover
  Soft-dead classification.
---

# Skill: Lean Facade Review

## Goal

New domain models often grow **thin wrappers** that feel “redundant” in review:

- `def title; payload['title']; end` on a JSON/jsonb column
- `def foo?; kind == KIND_FOO; end` used only in one view
- `I18n.t('a.b', default: "ハードコード…")` when `a.b` already exists in locale files
- Helper maps `source → badge` while the model also exposes `source_foo?` for icons

This skill tells reviewers **what is fine to keep**, **what is Soft-dead**, and **what comment severity to use** — without forcing Presenter/STI for every v1 feature.

## When to use

- Diff adds model methods that only read a hash/JSON column or compare to a constant
- Diff adds `I18n.t(..., default: "…")` with user-facing copy in the default string
- View branches on many `record.foo?` while a helper already keys off the same column
- Peer review asks “có nên xóa method / gọi trực tiếp hash không?”

## Do NOT use alone when

- Real domain logic (fallback messages, normalization, authz) lives in the method → keep; review the logic, not “thinness”
- Full CRUD cutover / orphan routes → `feature-cutover-orphan-review`
- Duplicate *business* logic across layers → `dry-duplication-scan`

---

## Decision table

| Pattern | Prefer | Review severity |
|---------|--------|-----------------|
| Passthrough getter used in **views** | Keep method **or** framework store accessor; **do not** push `hash['key']` into templates | 🔵 skip / optional `store_accessor` suggestion |
| Passthrough getter + real logic (`present?`, `Array()`, I18n fallback) | Keep method | skip |
| Writer/service already uses `hash['key']` | OK to leave; optional unify later | 🔵 |
| Predicate `col == CONST` used in **one** view; helper already maps `col` | Move branch into helper **or** `case col`; drop unused predicates | 🔵 nit |
| Predicate with **zero** `app/` callers | Soft-dead — delete or wire | 🔵 / Soft-dead |
| `I18n.t(key, default: "locale copy")` and **key exists** in locale files | Drop `default:`; keys are source of truth | 🔵 nit |
| `I18n.t` missing key → reviewer invents “must add i18n” without grepping locales | **False positive** — grep locales first | do not comment |
| Interpolated **master-data** labels inside an i18n template | Template structure = i18n; label values may stay domain language | skip |

---

## Hard rules (reviewer)

1. **Grep locales before “missing i18n”** — if `I18n.t('a.b.c')` and `a.b.c` exists in all required locale files, do **not** ask to “add i18n”; only flag redundant hard-coded `default:`.
2. **Views beat raw hash access** — `record.title` in ERB is better than `record.content['title']` for readability/typos. Thin getters for that purpose are **not** bloat by default.
3. **Logic stays in the method** — if removing the method forces copying I18n/Array/fallback into the view, **keep the method**.
4. **UI mapping lives in one place** — prefer helper (or one `case`) for badge/icon/label keyed by the same column; avoid model predicates **and** helper maps for the same concern.
5. **Don’t demand Presenter/STI** on v1 unless DoD requires typed payloads or multiple tables.
6. **Severity ceiling** — these findings are 🔵 nit or 🟡 suggestion max unless they break locale/EN UX or hide a missing key in production.

---

## Audit flow

### 1) Classify each new method

| Method | Body | Call sites (`app/` + views) | Class |
|--------|------|-------------------------------|-------|
| `#title` | `payload['title']` | view ×2 | Keep / store accessor |
| `#message` | I18n + fallback | view + specs | Keep (logic) |
| `#foo?` | `== CONST` | 0 | Soft-dead |
| `#bar?` | `== CONST` | view only; helper has map | Consolidate |

### 2) I18n `default:` check

```bash
# Key used in code
rg -n "I18n\.t\(\s*['\"]your.key" app/

# Key present in locales?
rg -n "your:" config/locales
```

| Result | Action |
|--------|--------|
| Key in all required locales + `default:` hardcode | Nit: remove `default:` |
| Key missing in a required locale | 🟠/🟡: add locale entry (real i18n gap) |
| Key present; reviewer wants “more i18n” on master-data interpolation | Skip |

### 3) Predicate vs helper

If helper already does `MAP.fetch(record.kind)` for label/badge:

- Icon/`if` branches on `record.kind_foo?` → suggest helper method or `case record.kind`
- Leave model predicates only if reused across **≥2** non-view layers

---

## Paste-ready comment shapes

**Redundant I18n default (keys exist):**

```markdown
🔵 Drop hard-coded I18n `default:` when locale keys already exist
📍 File: path (lines X–Y)

nit: `I18n.t('…')` already has locale entries. The `default: "…"` string only runs if the key is missing and can mask a missing translation. Remove `default:`; keep the locale files as source of truth.
```

**Thin predicates / consolidate UI:**

```markdown
🔵 Column predicates are UI-thin — consolidate with helper
📍 File: path (lines X–Y)

nit: `foo?` / `bar?` only compare a column to constants. A helper already maps that column for label/badge. Prefer one helper/`case` for icons too; delete predicates with zero app callers. Optional follow-up — not a merge blocker.
```

**Passthrough getters (usually skip):**

```markdown
🔵 Optional: store accessor for JSON passthrough getters
📍 File: path

nit: View-facing passthrough getters are fine. Optional cleanup: framework store accessor for pure keys; keep methods that add Array()/I18n/fallback. Do not switch templates to raw hash access.
```

---

## Anti-patterns (reviewer)

| Don’t | Do |
|-------|----|
| “Delete all getters; call hash in ERB” | Keep view API; optional store accessor |
| “Must add i18n” without grepping locale files | Grep first; maybe only drop `default:` |
| “Must extract Presenter” for 2 partials | YAGNI unless DoD |
| Inflate unused `baz?` to 🟠 | Soft-dead / 🔵 |

---

## Pairs with

| Skill / rule | When |
|--------------|------|
| `feature-cutover-orphan-review` | Soft-dead locale/predicate after cutover |
| `dry-duplication-scan` | Same mapping copied in model + helper + JS |
| `clean-code.mdc` | Dead code; Ruby I18n `default:` note |
| `pr-review-comments` | nit vs suggestion ceiling |
