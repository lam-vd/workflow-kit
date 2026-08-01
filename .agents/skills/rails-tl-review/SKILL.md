---
name: rails-tl-review
description: "ai-housemaker Stage 8 TL pass — scan staged Rails/Hotwire/Pundit/tenant diffs for architecture, security, performance, and testing issues that matter. Skip nitpicks without architecture/security impact. Used by /review-staged when profile ai-housemaker is active. Outputs TL Summary (Critical / Suggestions / Praise) that maps onto code-review severity (🔴🟠 block READY)."
---

# Skill: Rails TL Review (ai-housemaker)

> **VI**: Checklist Technical Lead cho staged diff ai-housemaker. Chỉ flag vấn đề ảnh hưởng kiến trúc / bảo mật / perf / correctness. Nitpick (naming preference, comment style) → bỏ qua trừ khi chắn review.

## When / Khi nào

- `/review-staged` với **ai-housemaker** profile (auto hoặc `(ai-housemaker)`).
- Chạy **sau** Phase 1 context + đọc `code-review` skill; **trước** Full report.
- song song với `ai-housemaker-review-checklist` (checklist feature UI/ActiveStorage vẫn áp dụng).

**Scope:** `git diff --cached` primarily; use PR-branch context only for related regressions.

## Mindset

| Do | Don't |
|----|--------|
| Fat controller / missing service or query object | Block on “I prefer X” without risk |
| Tenant leak / missing Pundit | Rename-only nits |
| N+1, missing transaction on multi-write | Micro-formatting |
| Turbo frame / Stimulus breakage | Extra abstraction “for future” |

Lean: `.cursor/rules/karpathy-guidelines.mdc` §2–§3.

---

## Checklist (scan staged paths only)

Mark each applicable item Pass / Fail / N/A. Fail → finding with severity below.

### 1. Architecture & Design

| Check | Fail pattern | Severity |
|-------|--------------|----------|
| Business logic in service (`app/services`), not fat controller | Multi-step domain logic / branching in controller action | 🟠 |
| Complex SQL / multi-table filters → `app/queries` | Long scopes / raw SQL inlined in controller or unrelated model | 🟡→🟠 if untestable / leak risk |
| Guard clauses / early returns | Deep nested `if/else` that hides happy path | 🟡 (🟠 if bugs likely) |
| New abstraction justified | Parallel service wrapping one-liner already elsewhere | 🟠 YAGNI |

Canonical: `when-to-extract-service`, `when-to-use-query-object`, `refactor-early-returns`, `anti-patterns`.

### 2. Security & Tenant Scoping

| Check | Fail pattern | Severity |
|-------|--------------|----------|
| Reads/writes tenant-scoped (`Current.tenant`, `TenantScoped`, `policy_scope`) | Unscoped `Model.find` / `create` across tenants | 🔴 |
| `authorize` / Pundit before mutate + after_action verify | Missing policy on create/update/destroy | 🟠→🔴 |
| Brakeman / VerbConfusion / obvious SQLi XSS | `request.get?` only on GET+POST; string-interpolated SQL; raw HTML | 🟠→🔴 |

Canonical: `tenant-scoping.mdc`, `policies.mdc`, `brakeman.mdc`, `idor-prevention` skill.

### 3. Performance & Database

| Check | Fail pattern | Severity |
|-------|--------------|----------|
| N+1 — `includes` / `preload` / `eager_load` where associations looped in view/serializer | Association access in loop without eager load | 🟠 |
| Multi-record / attach+scalar writes in **one** `transaction` | Split saves that leave orphan state | 🟠 (🔴 if data corruption) |
| Migrations follow `strong_migrations` | Blocking index / dangerous change without safe pattern | 🟠→🔴 |

Canonical: `n-plus-one-prevention`, `database-safety`, `migrations.mdc`, audited-active-storage (avatar path).

### 4. Frontend & Views (Hotwire / Stimulus / ERB)

| Check | Fail pattern | Severity |
|-------|--------------|----------|
| Turbo Frame / Stream targets correct; no accidental full-page reload | Wrong `turbo_frame_tag` / missing frame on form | 🟠 `[integration]` |
| Nested Preline: teleport + 422 replaces modal id + backdrop cleanup + explicit bool strings | Replace parent frame on fail; `open-value=""`; open without stripping backdrops | 🟠 `[integration]` |
| Stimulus owns DOM / file preview; clear file input on 削除 | Logic in ERB script tags; cancel leaves file selected | 🟠 (see checklist) |
| i18n JA for new user-facing copy | Hardcoded EN/VI in new UI | 🟡→🟠 |

Canonical: `hotwire-integration-patterns.mdc`, `stimulus-file-input-preview.mdc`, `views.mdc`, integration-regression-review.

### 5. Testing & Quality

| Check | Fail pattern | Severity |
|-------|--------------|----------|
| Request specs = status / redirect / flash / DB / jobs / `media_type` — **HARD BAN** DOM/copy/CSS/Stimulus in body | `include('閲覧済')`, `I18n.t` in body, BEM `.scan`, option-card markup | 🟠 → BLOCKED |
| Related specs run & pass (functional verification) | Staged behavior untested / failing | 🟠→ BLOCKED |
| RuboCop / ERBLint clean on staged | Lint errors | 🟠 |
| Method complexity smell | Huge method doing 3 jobs | 🟡 unless correctness risk → 🟠 |

Canonical: `rspec-best-practices`, `ai-housemaker-rspec` / `rspec-patterns` (**HARD BAN**), `erb-rubocop-lint`, review-linters.  
UI polish / autosave scroll / hover → Manual QA — **never** “fix” by adding request body expects.

---

## Severity map → Stage 8 verdict

| TL label | code-review severity | READY |
|----------|----------------------|-------|
| 🚨 Critical Issues | 🔴 P0 / 🟠 P1 | **BLOCKED** |
| 💡 Suggestions | 🟡 P2 (architecture/perf only) | READY OK |
| Nitpicks skipped | (omit) or 🟢 only if must note | READY OK |
| ✅ Praise | Positive observations | required ≥1 from this pass if diffs substantial |

Every 🔴/🟠 finding still needs code-review structure: Hiện tượng / Likelihood / Ảnh hưởng / Gợi ý (+ Cách tái hiện for functional/UI).

---

## TL Summary block (emit inside Full report)

Place **after** metadata table / before or after Findings — keep Full report sections intact:

```markdown
## TL Summary (rails-tl-review)

### 🚨 Critical Issues
- … (or *None*)

### 💡 Suggestions & Refactoring
- … (architecture/perf only; or *None*)

### ✅ Praise
- …
```

Then continue: Findings (detailed), Intent & Coverage, Verdict READY/BLOCKED.

---

## Path skip matrix

| Staged paths | Skip sections |
|--------------|---------------|
| No `app/views` / JS / CSS | §4 Hotwire detailed; still note N/A |
| No `db/migrate` | Migration row N/A |
| Docs-only | Architecture light; focus Intent & Coverage |
| Specs-only | Focus §5 + related app code in PR branch if present |
