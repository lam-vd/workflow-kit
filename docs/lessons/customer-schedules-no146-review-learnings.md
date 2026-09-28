# Review learnings — Customer schedules (No.146)

> **Source:** PR `feat/customer-details-schedule-management` — Copilot, member dev (phuoctran), Stage 8 self-review, QA manual.  
> **VI:** Kinh nghiệm review thực tế cho feature Hotwire sidebar panel + nested CRUD. Dùng để nâng rules/skills — không copy-paste logic feature vào shared module.

---

## Executive summary / Tóm tắt

| Theme | Reviewer signal | Rule/skill upgraded |
|-------|-----------------|---------------------|
| Thin query service | YAGNI — 1-line wrapper | `housemaker-services`, `dry-duplication-scan` |
| Lock race | check-then-act | `housemaker-services` transaction-patterns |
| Assignee scope | QA + member: tenant-wide, not customer history | Feature DDD + boundary review |
| Datetime vs date | QA: inactive assignee at “today + past time” | `edge-case-boundary-review` TIME-* |
| Timezone | Dev VN vs prod JST | `language-boundaries` + TIME-JST-01 |
| Turbo scroll | Load-more + stream replace jump | `hotwire-integration-patterns` TURBO-SCROLL-* |
| Cumulative load-more | Cap height at PER items, not fixed rem | `paginated-list-patterns` PAGE-CUMULATIVE-* |
| due_at validation | Year > 4 digits; blank vs invalid message | TURBO-VALID-02, controller early reject |
| Request spec layer | Toast copy in body vs HARD BAN | `rspec-patterns`, `ai-housemaker-review-checklist` |
| Dead code after delete | AssigneeOptionsQuery orphan grep | `deadcode-ui-migration-review` |

---

## 1. Copilot & member dev feedback → action

### 1.1 Race on lock (member — phuoctran) — **must fix**

**Comment pattern:** “Check-then-act between `actions_locked?` and update — concurrent request can pass check then both mutate.”

**Fix:** `schedule.with_lock { re-check actions_locked?; mutate }` in Update/Destroy.

**Prevention (implement/review):**

- Any “read status → decide → write” on same row → **pessimistic lock** or optimistic `lock_version`.
- Add regression spec: simulate concurrent state change inside `with_lock` block.

**Skill:** `housemaker-services/references/transaction-patterns.md` — add “status gate under lock”.

---

### 1.2 Inactive assignee scope (member + QA)

**Initial implementation:** Active users + inactive users who had schedules **with this customer**.

**Reviewer/QA intent:** Dropdown = **all active + inactive in tenant** (exclude `pending` only). Not limited to prior schedule assignees.

**Fix:** `TenantUser.assignable_for_schedules` scope; remove `Schedules::AssigneeOptionsQuery`.

**Lesson:** When dropdown data scope is ambiguous, **grep DDD Decision Log** before shipping query that encodes wrong business rule.

---

### 1.3 Admin-only destroy (QA)

**Rule:** Delete button + `SchedulePolicy#destroy?` → admin only (not manager/assignee).

**Review check:** Policy spec + request spec + UI `policy(schedule).destroy?` gate — all three layers.

---

### 1.4 AssigneeOptionsQuery removal (self-review / DRY)

**Anti-pattern:** Service that only delegates to `tenant_users.active.order(...)` + merge inactive from SQL.

**Preferred:** Model scope on `TenantUser` when logic is status filter + order (no multi-step domain).

**Gate:** `dry-duplication-scan` — if new `*Query` has ≤1 consumer and no transaction/lock, prefer scope or inline in controller loader.

---

## 2. QA manual findings → patterns

### 2.1 Inactive assignee: date vs datetime (TIME-CMP-01)

**Bug:** Rule used **calendar day** (`due_date < today`) — user could not pick inactive assignee for “today 10:00” when now is 14:00.

**Correct rule:** `due_at < Time.zone.now` (full datetime in **app timezone**).

**JS:** Compare datetime-local; accept optional `:SS`; do not compare only first 10 chars of date.

---

### 2.2 Timezone (TIME-JST-01)

**Config:** `config.time_zone = 'Asia/Tokyo'`.

**Dev trap:** Browser `datetime-local` uses **local OS timezone**; server parses as JST. Dev in VN sees “wrong” enable/disable for inactive assignee.

**QA note:** Compare against `Time.zone.now` in rails runner; document in Manual QA matrix.

---

### 2.3 due_at year validation (TURBO-VALID-02)

**Bug:** User could enter year with >4 digits (browser quirk / manual input).

**Fix (3 layers):**

| Layer | Enforcement |
|-------|-------------|
| HTML | `min` / `max` on `datetime-local` |
| JS | `setCustomValidity` + block submit |
| Server | `Schedule.valid_due_at_input?` + range validation |

**P2 trap:** Controller returned `nil` for invalid parse → model said **blank**, not **invalid**. Fix: early `respond_invalid_due_at` with dedicated locale key.

---

### 2.4 Panel scroll + load-more (TURBO-SCROLL-01/02)

**Symptoms:**

1. After load-more, page scroll jumps to top.
2. After save/delete (turbo_stream), same jump.
3. Fixed `max-height` rem looks empty when few items.

**Root cause:** Turbo **replaces** panel DOM → `#syncListScroll` cleared `maxHeight` → list expanded to full N items → layout reflow → `main` scroll reset.

**Fix pattern:**

- ≤ `PER` items: **natural height** (no cap).
- \> `PER` items: cap height to height of first `PER` items; inner scroll.
- **Both** `turbo:before-frame-render` (load-more) **and** `turbo:before-stream-render` (CRUD success) must capture `list.clientHeight` + scroll positions **before** replace, chain `event.detail.render`, restore in `rAF`.
- Guard: only `customer-schedules-panel` frame / stream target.

---

### 2.5 Cumulative pagination (PAGE-CUMULATIVE-01)

**Pattern:** `limit = page * PER`, append via load-more (not offset replace).

**Review checks:**

- `has_more` via `limit + 1` peek.
- Hidden `page` field on mutate forms to preserve panel page.
- Spec: spy `ForCustomerQuery` with `page: 2`, not `data-page` in response body.

---

## 3. Request spec layer lessons

### HARD BAN reminder

`spec/requests/**` must not assert UI copy, BEM, Stimulus attrs.

### Borderline: turbo toast message

Asserting `response.body.include?(I18n.t('...invalid_due_at'))` tests **toast HTML**, not HTTP contract.

| Prefer | Avoid |
|--------|--------|
| `have_http_status(:unprocessable_content)` + no DB change | `include('期限切れ')` for layout |
| Service/controller spec for error message list | Scanning option card markup |
| Spy `ForCustomerQuery.call(page: 2)` | `data-page="2"` in body |

**If toast copy must be tested:** request spec OK for **one** dedicated example per error key, or move to service spec that returns `errors` array.

---

## 4. Stage 8 review checklist additions

When reviewing similar features (sidebar panel + modal CRUD + cumulative load-more):

- [ ] Lock race on status-gated mutate?
- [ ] Dropdown scope matches Decision Log (tenant vs feature-scoped)?
- [ ] Datetime rules use `Time.zone`, not date-only, when time matters?
- [ ] Validation enforced server-side (not maxlength-only)?
- [ ] Invalid input → **specific** error, not generic blank?
- [ ] Turbo frame **and** stream replace preserve scroll when list capped?
- [ ] List height: natural ≤ PER, cap + scroll > PER?
- [ ] Removed service/query — grep zero call sites?
- [ ] Request specs lean (`create_list` count matches PER+1, not 26 for PER=10)?

---

## 5. Copilot triage hints (No.146)

| Copilot suggestion | Verdict | Reason |
|--------------------|---------|--------|
| Add pessimistic lock | **Fix** | Valid race |
| Extract assignee query service | **Skip** | Prefer model scope |
| Use `default_scope` for tenant | **Skip** | Use `TenantScoped` |
| Assert DOM in request spec | **Skip** | HARD BAN |
| Add `maxlength` only for due_at | **Partial** | Need server rule too |

---

## 6. Files updated from this doc

| Artifact | Change |
|----------|--------|
| `hotwire-integration-patterns.mdc` | TURBO-SCROLL-01, TURBO-SCROLL-02, TURBO-VALID-02 |
| `paginated-list-patterns.mdc` | PAGE-CUMULATIVE-01, PAGE-SCROLL-01 |
| `ai-housemaker-review-checklist/SKILL.md` | Incident row No.146 |
| `copilot-review-triage/SKILL.md` | No.146 decision table |
| `housemaker-services/SKILL.md` | Thin query / scope guidance |
| `edge-case-boundary-review/SKILL.md` | TIME-JST-01, TIME-CMP-01 pointers |

---

## Decision log reference (locked)

- Inactive assignee allowed iff `due_at < Time.zone.now`
- Dropdown: all tenant active + inactive (no pending)
- Destroy: admin only
- Update/Destroy: `with_lock` + re-check `actions_locked?`
- Panel page preserved on mutate via hidden `page`
