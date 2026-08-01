---
name: edge-case-boundary-review
description: "Stage 8 supplement — systematic edge case and boundary review of a staged diff. For every boundary the diff touches, force an explicit answer for INSIDE the boundary, AT the boundary, and OUTSIDE it (reject / clamp / no-op / redirect). Catalog with stable IDs (EDGE-CARD/NIL/NUM/STR/IDX/TIME/STATE/CONC/AUTHZ/EXT) covering 0-1-N-N+1, nil vs empty vs zero, min/max/precision/overflow, string length + multibyte, off-by-one + inclusive/exclusive ranges, timezone/DST/end-of-day/month-end, double submit + races + idempotency, illegal state transitions, scope edges, external failure. Also checks that the same limit is enforced consistently across DB / model / controller / JS. Pagination boundaries defer to paginated-list-patterns.mdc."
---

# Skill: Edge Case & Boundary Review

## Goal

Diff review usually verifies the happy path. Production breaks at the **edges**.

For every boundary in the staged diff, the review must answer three questions — not one:

| Position | Question |
|----------|----------|
| **Inside** (`B-1`) | Does the normal case still work right up to the limit? |
| **At** (`B`) | Is the limit inclusive or exclusive? Which one did we intend? |
| **Outside** (`B+1`) | What *exactly* happens — 422? clamp? no-op? redirect? silent truncation? |

> **VI**: Không được kết luận "OK" khi mới chỉ thử giá trị ở giữa dải. Phải nêu rõ hành vi **trong biên**, **tại biên**, và **ngoài biên**.

An unanswered "Outside" is a finding, not a gap in the review.

## Use when (Stage 8)

Mandatory when the staged diff contains any of:

- a comparison operator (`<`, `<=`, `>`, `>=`), a range (`..`, `...`), `limit`, `first`, `last`, `[n]`
- a validation (`length`, `numericality`, `presence`, `inclusion`), or a DB constraint / column limit
- a date, time, duration, expiry, or timezone conversion
- a loop, `each_slice`, `in_batches`, counter, or index arithmetic
- a status/enum transition, or a `update`/`create` that another actor could run concurrently
- money, percentage, rounding, or division

Skip only for pure copy/CSS/locale-text diffs — and say so explicitly.

## Do NOT duplicate

- **Pagination / page overflow / search-narrowing** → `.cursor/rules/paginated-list-patterns.mdc` (`PAGE-*`). Cite its IDs; do not restate them here.
- **Text/chip visual overflow** → `css-safe-text-overflow` (`CSS-OVERFLOW-01`).
- **Turbo/URL/history state** → `integration-regression-review`.
- **Stimulus autosave races** → `stimulus-turbo-autosave` (`TURBO-AUTOSAVE-*`); this skill covers the server-side race.

---

## Phase 1 — Boundary inventory (extract from the diff)

One row per boundary the diff introduces or moves. Empty table ⇒ prove the diff has no boundary.

| # | Input / value | Declared limit | Inside (`B-1`) | At (`B`) | Outside (`B+1`) | Enforced at | Source of truth |
|---|---------------|----------------|----------------|----------|-----------------|-------------|-----------------|
| B1 | `title` | 255 chars | saves | saves (inclusive) | 422 + field error | model + DB | migration |
| B2 | `page` | ≥1 | — | page 1 | coerce to 1 | controller | `PageParam` |

**Rule:** "Outside" must name a concrete observable behavior. `undefined` / `probably validated` / `should be fine` = 🟠 finding.

---

## Phase 2 — Enforcement consistency (highest-yield check)

The same limit declared in several layers with different values is a silent data bug.

| Layer | Value found | Matches source of truth? |
|-------|-------------|--------------------------|
| DB column / constraint | `varchar(255)` | ✅ |
| Model validation | `maximum: 250` | ❌ mismatch |
| Controller / strong params | — | n/a |
| JS / HTML `maxlength` | `300` | ❌ mismatch |
| Locale error message | "255文字以内" | ❌ mismatch |

Severity guide:
- Client-only enforcement (JS/`maxlength`) with **no** server rule → 🔴 (bypassable).
- Server stricter than DB → 🟡 (safe but message may lie).
- DB stricter than server → 🟠 (`ActiveRecord::ValueTooLong` 500 instead of 422).
- Error message states a different number than the rule → 🟡.

Cross-reference: if the diff added a second copy of an existing rule → `DUP-VALID-01` in `dry-duplication-scan`.

---

## Phase 3 — Boundary catalog (sweep; mark PASS / FAIL / N/A + evidence)

### Cardinality — `EDGE-CARD-*`

| ID | Case | Typical failure |
|----|------|-----------------|
| `EDGE-CARD-01` | **0 items** — empty collection | Empty state missing; `.first.name` → NoMethodError; `sum / count` → ZeroDivisionError |
| `EDGE-CARD-02` | **1 item** | Pluralization, "and" joins, delete-last-row leaves broken UI |
| `EDGE-CARD-03` | **N vs N+1** at a `limit` | Truncated list with no "more" indicator; batch loop drops the tail |
| `EDGE-CARD-04` | **Exactly the limit** | Off-by-one between `limit` and `count` |
| `EDGE-CARD-05` | **Duplicates in the collection** | `uniq` assumed but not applied; join fan-out inflates counts |

### Nullability — `EDGE-NIL-*`

| ID | Case | Typical failure |
|----|------|-----------------|
| `EDGE-NIL-01` | `nil` vs `""` vs `0` vs `false` treated as the same | `blank?` swallows a legitimate `0` / `false` |
| `EDGE-NIL-02` | `nil` in comparison / sort / arithmetic | `nil` can't be coerced; `NULL` sorts unexpectedly |
| `EDGE-NIL-03` | Optional association absent | `record.owner.name` on a nullable belongs_to |
| `EDGE-NIL-04` | Param key missing vs present-but-empty | "clear the field" silently ignored |

### Numeric — `EDGE-NUM-*`

| ID | Case | Typical failure |
|----|------|-----------------|
| `EDGE-NUM-01` | min / max accepted value | Exclusive vs inclusive confusion |
| `EDGE-NUM-02` | zero and negative | Negative quantity/price accepted |
| `EDGE-NUM-03` | Float/rounding on money & percentages | ¥1 drift; `round` vs `floor` on tax |
| `EDGE-NUM-04` | Division by zero / empty average | 500 on an empty dataset |
| `EDGE-NUM-05` | Column range (`integer` vs `bigint`), overflow | `RangeError` on large ids/amounts |
| `EDGE-NUM-06` | String → number coercion of params | `"abc".to_i == 0` silently becomes a valid value |

### String — `EDGE-STR-*`

| ID | Case | Typical failure |
|----|------|-----------------|
| `EDGE-STR-01` | Length at limit, multibyte (JA) | DB counts bytes, validation counts chars |
| `EDGE-STR-02` | Leading/trailing whitespace, full-width space | Uniqueness bypass; "empty" value passes `presence` |
| `EDGE-STR-03` | Emoji / 4-byte UTF-8 | `utf8` vs `utf8mb4` truncation |
| `EDGE-STR-04` | Special chars in search / LIKE (`%`, `_`, `\`) | Unescaped wildcard changes results |

### Index & range — `EDGE-IDX-*`

| ID | Case | Typical failure |
|----|------|-----------------|
| `EDGE-IDX-01` | Off-by-one in slicing / counters | Last element dropped or repeated |
| `EDGE-IDX-02` | `..` vs `...` (inclusive vs exclusive) | End value silently excluded |
| `EDGE-IDX-03` | Index beyond length | `nil` propagates instead of raising early |
| `EDGE-IDX-04` | Reverse/empty range (`from > to`) | Returns empty instead of an error |

### Date & time — `EDGE-TIME-*`

| ID | Case | Typical failure |
|----|------|-----------------|
| `EDGE-TIME-01` | Timezone: stored UTC, displayed Asia/Tokyo | Record shows on the wrong day near midnight |
| `EDGE-TIME-02` | Start/end of day on a filter | `<= end_date` excludes same-day records with a time component |
| `EDGE-TIME-03` | Inclusive vs exclusive end date | "until 3/31" misses 3/31 |
| `EDGE-TIME-04` | Month-end / leap day / year boundary | `+1.month` from Jan 31; Feb 29 |
| `EDGE-TIME-05` | Expiry exactly at the boundary instant | `>` vs `>=` decides valid/expired |
| `EDGE-TIME-06` | `Date` compared to `Time`/`DateTime` | Implicit midnight shifts the comparison |
| `EDGE-TIME-07` | Past/future values, clock skew, DST | Negative duration; scheduled job fires twice or never |

### Concurrency & idempotency — `EDGE-CONC-*`

| ID | Case | Typical failure |
|----|------|-----------------|
| `EDGE-CONC-01` | Double submit / double click | Two records; two emails |
| `EDGE-CONC-02` | Read-modify-write race (counter, balance, status) | Lost update; needs lock or atomic update |
| `EDGE-CONC-03` | Uniqueness validated in app only | Race passes validation, DB has no unique index |
| `EDGE-CONC-04` | Record deleted/changed between page render and submit | 404/500 instead of a friendly stale-data message |
| `EDGE-CONC-05` | Job retry / at-least-once delivery | Non-idempotent side effect repeated |

### State machine — `EDGE-STATE-*`

| ID | Case | Typical failure |
|----|------|-----------------|
| `EDGE-STATE-01` | Illegal transition attempted directly (URL/API) | Skips required intermediate state |
| `EDGE-STATE-02` | Re-applying the current state | Duplicate side effects (notification resent) |
| `EDGE-STATE-03` | Terminal state mutated | Completed/cancelled record edited |
| `EDGE-STATE-04` | State changed by another actor mid-flow | UI acts on a stale status |

### Scope & authorization edges — `EDGE-AUTHZ-*`

| ID | Case | Typical failure |
|----|------|-----------------|
| `EDGE-AUTHZ-01` | Id belonging to another tenant / salon | IDOR — scope applied on read but not on write |
| `EDGE-AUTHZ-02` | Role exactly at the permission boundary | Staff can reach a master-only action |
| `EDGE-AUTHZ-03` | Nested resource id mismatched with parent | `/parents/1/children/999` from another parent |

### External dependencies — `EDGE-EXT-*`

| ID | Case | Typical failure |
|----|------|-----------------|
| `EDGE-EXT-01` | Timeout / 5xx / empty response | Unhandled exception surfaces to the user |
| `EDGE-EXT-02` | Partial success (2 of 3 succeeded) | No compensation; inconsistent state |
| `EDGE-EXT-03` | Response shape changed / field missing | `nil` propagates into persisted data |

---

## Phase 4 — Test & QA mapping

Each **FAIL** and each 🔴/🟠 boundary must map to one of:

| Evidence type | Acceptable form |
|---------------|-----------------|
| Automated | `rspec path:line` output, exit code |
| Manual QA | Preconditions + steps + expected + actual |
| Not run | Explicit reason — counts as **unverified**, not PASS |

`N/A` requires a reason (`no date logic in diff`). Blank cell = incomplete review.

---

## Severity mapping

| Situation | Severity |
|-----------|----------|
| Outside-boundary behavior undefined on a write path | 🔴 P0 |
| Client-only limit, no server enforcement | 🔴 P0 |
| Boundary produces 500 instead of a validation error | 🟠 P1 |
| Concurrency/idempotency gap on a side-effecting action (`EDGE-CONC-01/05`) | 🟠 P1 |
| Layer limits mismatch (DB vs model vs JS) | 🟠 P1 |
| Inclusive/exclusive mismatch on a date filter | 🟠 P1 |
| Boundary correct but untested | 🟡 P2 |
| Error message states the wrong number | 🟡 P2 |

Findings are emitted through the `code-review` skill's finding structure (Hiện tượng / Likelihood / Ảnh hưởng / Cách tái hiện / Gợi ý) — this skill only supplies **which** edges to check and their IDs.

---

## Output template (goes into the Stage 8 report)

```markdown
## 1d. Edge case & boundary sweep

### Boundary inventory
| # | Value | Limit | Inside | At | Outside | Enforced at |
|---|-------|-------|--------|----|---------|-------------|

### Enforcement consistency
| Layer | Value | Match |
|-------|-------|-------|

### Catalog sweep
| ID | Case | Result | Evidence |
|----|------|--------|----------|
| `EDGE-CARD-01` | Empty list | PASS | `rspec spec/requests/x_spec.rb:42` |
| `EDGE-TIME-03` | Inclusive end date | FAIL | 3/31 record missing → finding 1.2 |
| `EDGE-CONC-01` | Double submit | NOT RUN | no idempotency key; manual QA pending |

**Deferred to other catalogs:** `PAGE-*` (paginated-list-patterns), `CSS-OVERFLOW-01`.
```

---

## Hard rules

- Every boundary in the diff gets a row with **all three** positions filled (inside / at / outside).
- "Outside" answered with a concrete behavior, never "should be validated".
- `PASS` requires evidence (test command or QA steps). No evidence ⇒ `NOT RUN`.
- `N/A` requires a one-line reason.
- Never restate `PAGE-*` cases here — cite `paginated-list-patterns.mdc`.
- A limit enforced only in JS/HTML is 🔴 regardless of how the UI behaves.
- Date filters: state explicitly whether the end bound is inclusive, and in which timezone.

## Quick checklist (30-second pass)

- [ ] Listed every comparison / range / limit / date / counter in the diff
- [ ] For each: inside, at, outside answered
- [ ] Same limit checked across DB / model / controller / JS / locale
- [ ] 0 / 1 / N / N+1 considered for every collection
- [ ] nil vs empty vs zero distinguished where it matters
- [ ] Timezone + inclusive/exclusive stated for every date filter
- [ ] Double submit / retry considered for every side-effecting action
- [ ] Illegal state transition attempted from URL/API, not just UI
- [ ] Pagination edges delegated to `PAGE-*`, not re-invented

## Pairs with

| Skill / rule | When |
|--------------|------|
| `code-review` | Finding structure + report host |
| `functional-verification-review` | Turning FAIL rows into runnable tests / QA scripts |
| `paginated-list-patterns.mdc` | Any list/search/pagination boundary |
| `integration-regression-review` | Boundary that only appears through Turbo/URL state |
| `modal-detail-canonical-url` | Canonical list detail modal URLs / deep-link / filter restore (TURBO-MODAL-URL-*) |
| `dry-duplication-scan` | `DUP-VALID-01` — the same rule duplicated with different limits |
| `writing-ddd` | Edge cases that should have been in the spec → back to Stage 3 |
