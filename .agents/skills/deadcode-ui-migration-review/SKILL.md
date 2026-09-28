---
name: deadcode-ui-migration-review
description: "Stage 7–8 skill for auditing orphaned UI after a screen migration (full-page → modal/Turbo frame, old partial → new layout). Classifies Dead-sure / Fallback-only / Soft-dead payload / Shared-keep, greps call sites before delete, and blocks silent full-page removal without redirect/fallback. Use when reviewing or cleaning deadcode after UI replacement (e.g. proposals#show → detail modal), deleting unused partials/helpers/JS, or /review-staged on migration PRs."
---

# Skill: Deadcode UI Migration Review

## Goal

When product UI moves (full page → modal, sidebar form → footer toggle, old partial → new frame), leftover views/helpers/services often stay **wired as fallback** or become **true orphans**. This skill forces a call-site audit before delete, and prevents “delete show page” without a safe non-modal path.

Canonical incident (ai-housemaker, Jul 2026): `GET /proposals/:id` full-page stack (`show.html.erb`, `_media_hero`, `_header`, `_sidebar`, `_description`, `_proposal_history_bar`, `_status_memo_frame`) replaced by `_detail_modal_frame`, while BE payloads/`RelatedForCustomerQuery` still had live consumers.

## When to use

- User or PR says: deadcode / unused partial / “UI mới không dùng nữa” / full page replaced by modal
- Stage 7 cleanup after shipping a modal/Turbo-frame detail
- Stage 8 `/review-staged` when diff deletes views **or** leaves old show/index path beside a new modal
- Parallel to `structural-change-analysis` (plan) and `shared-abstraction-safety` (don’t break other screens)

## Do NOT use alone when

- Pure domain refactor with no UI surface change → `structural-change-analysis`
- CRUD UI removed / domain data kept / job·enqueue·channel orphans → **also** run `feature-cutover-orphan-review` (stack-agnostic cutover; this skill stays UI-migration focused)
- Diff-only style review → `code-review`
- Hotwire wiring bugs (stale frame, backdrop) → `integration-regression-review`

---

## Hard rules

1. **Grep before delete** — 0 render/call sites in `app/` (+ Stimulus) before calling something Dead-sure.
2. **Fallback ≠ dead** — If controller still renders old template on non-frame GET, it is **Fallback-only** until product chốt modal-only **and** show redirects/404s.
3. **Shared query keep** — Same query/service used by another screen (e.g. customer tab) → **Shared-keep**; delete only the consumer-specific serializer/row mapper.
4. **Replace failure UX** — Removing a Turbo Frame partial requires rewriting 422/replace targets (toast / memo modal / detail frame), not leaving streams pointed at deleted DOM ids.
5. **HTML redirects** — After removing full-page show, `format.html { redirect_to … }` must not bounce to the deleted page.
6. **Specs are consumers** — Request specs asserting old labels/DOM keep the path “alive”; update or delete them in the same PR.
7. **Soft-dead payload is follow-up** — Hash keys built only for deleted views may remain until a dedicated prune; do not silently shrink shared payloads in a UI-delete PR unless specs/views are updated together.

---

## Audit flow (mandatory order)

### 1) Name the replacement surface

| Old (candidate dead) | New (live) | Entry points |
|----------------------|------------|--------------|
| e.g. `proposals/show` | e.g. `_detail_modal_frame` | meeting / history / customer links + Turbo-Frame header |

Write one sentence: *“Product opens X via Y; Z is leftover of old full page.”*

### 2) Trace the controller gate

Find how old vs new is chosen:

```ruby
# Pattern to look for
render_detail_modal_frame if turbo_frame_request_for?(FRAME_ID)
# else → default template (FALLBACK) or redirect (MODAL-ONLY)
```

Classify current gate:

| Gate | Meaning |
|------|---------|
| Conditional render + default template | Fallback-only stack still live |
| `unless modal…; redirect; return; end` + always modal partial | Modal-only (safe to delete old views) |
| Always old template | Migration incomplete |

### 3) Inventory candidates

List files/helpers/services **only** referenced by the old surface:

- Views/partials rendered solely from old template
- Helpers used only by those partials
- Services/serializers building rows **only** for old bars/sidebars
- JS methods defined but never called after UI rewrite
- i18n keys under the old namespace
- Failure paths (`turbo_stream.replace` target = old frame id)
- **Stylesheets / BEM blocks** tied only to deleted markup (see §3b)

### 3b) CSS / stylesheet audit (mandatory with HTML delete)

Deleting ERB without checking CSS leaves orphan rules (or false confidence that “no CSS existed”).

| Step | What to do |
|------|------------|
| Collect selectors | From deleted ERB: BEM roots (`meeting-proposal-…`, `proposal-history-…`), custom classes (`desired-area-row`), `data-*` styled in CSS |
| Grep stylesheets | `app/assets/stylesheets/**/*.css` (+ compiled builds only as hint) |
| Grep remaining views/JS | Same class names — Shared-keep if modal/history still uses them |
| Feature CSS file | If `@import "./foo.css"` and **all** selectors were only for deleted UI → delete file + remove import from `application.tailwind.css` |
| Tailwind-only UI | Utilities in `class="…"` with **no** custom CSS file → document “no stylesheet to delete” in audit report (do not invent a CSS cleanup) |
| Shared components.css | Call-site first; never strip a shared utility used elsewhere |

Classify CSS like HTML: **Dead-sure** (0 remaining call sites) / **Shared-keep** (modal or other screens) / **Soft-dead** (compiled leftover until rebuild — rebuild assets after source delete).

Add to checklist before merge:

- [ ] Grepped class/BEM names from deleted views against `app/assets/stylesheets`
- [ ] Removed orphan feature CSS + `@import` **or** explicitly noted Tailwind-only (no file)
- [ ] Modal/new UI CSS files kept (`proposal-history-detail.css`, `meeting-proposals.css`, …)

**ai-housemaker note (Jul 2026 show cutover):** full-page `proposals/show` + partials used **Tailwind utilities only** — no dedicated `proposals-show.css`. Modal CSS (`proposal-history-detail.css`, `meeting-proposals.css`, `proposal-memo.css`) is live — do not delete with show HTML.

### 4) Grep matrix (required)

For each candidate, grep **call sites** in:

- `app/views`, `app/controllers`, `app/helpers`, `app/services`, `app/queries`, `app/javascript`
- `spec/` (tests count as consumers for “still required”)

Fill:

| Symbol / file | Call sites (count + paths) | Class |
|---------------|----------------------------|-------|
| … | 0 in app/ | **Dead-sure** |
| … | only old show + specs | **Fallback-only** (or Dead after modal-only) |
| … | also customer/meeting/other | **Shared-keep** |
| … | built in payload, 0 view reads | **Soft-dead payload** |

### 5) Classify (use these labels)

| Class | Definition | Action |
|-------|------------|--------|
| **Dead-sure** | 0 production call sites | Delete in cleanup PR (+ specs/locales) |
| **Fallback-only** | Only non-modal / old template path | Do **not** delete until gate is modal-only + redirect; then delete with specs |
| **Soft-dead payload** | Service still builds keys; no view reads them | Optional follow-up prune; keep if BE reuse planned |
| **Shared-keep** | ≥2 product surfaces | Keep; delete only feature-specific wrapper |
| **Failure-orphan** | Stream/replace targets removed DOM | Rewrite to toast/modal failure **before** deleting partial |

### 6) Safe deletion checklist (modal-only cutover)

Before deleting Fallback-only stack:

- [ ] `show`/`edit` requires Turbo-Frame (or equivalent) else `redirect_to` known index (see houses/lands pattern)
- [ ] All product links open via modal host (`data-*-frame`, `proposal_detail_modal_link`, etc.)
- [ ] `format.html` redirects after create/update point away from deleted page
- [ ] Failure streams no longer target deleted frame ids
- [ ] Request specs: content assertions use frame headers; non-frame expects redirect
- [ ] Grep again after delete — no stale `render "…"` 

### 7) Output template (review / audit report)

```markdown
## Deadcode UI migration audit

**Replacement:** <old> → <new>
**Controller gate:** Fallback-only | Modal-only | Incomplete

### Findings

| Item | Class | Evidence (grep) | Action |
|------|-------|-----------------|--------|
| … | Dead-sure | 0 callers | Delete |
| … | Shared-keep | customers/detail_page_loader | Keep |
| … | Soft-dead payload | ShowPayload#:tags | Follow-up |

### Blockers
- [ ] None / [ ] Gate still Fallback-only — do not delete show stack

### Suggested PR split
1. Dead-sure only (partials/JS/helpers with 0 callers)
2. Modal-only cutover (redirect + delete Fallback stack + specs)
3. Optional payload prune (Soft-dead keys + specs)
```

Severity when used inside `/review-staged`:

| Finding | Severity |
|---------|----------|
| Deleted shared query still used elsewhere | 🔴 |
| Removed frame but 422 still replaces its id | 🟠 |
| Left full-page show as silent fallback after “modal-only” claim | 🟠 |
| Soft-dead payload keys unused | 🟡 / defer |
| Dead-sure file still in tree after cleanup PR | 🟠 |

---

## Rails / Hotwire heuristics (ai-housemaker)

| Signal | Likely class |
|--------|--------------|
| Partial only in `show.html.erb`; modal uses `_detail_modal_frame` | Fallback-only → Dead after cutover |
| `Related*Query` used by customer detail **and** old history bar | Shared-keep query; Dead-sure row mapper for bar only |
| `include_gallery: true` but views never read `proposal[:land_images]` | Soft-dead payload (+ wasted preload) |
| JS method defined, never called (`replacementMeta`) | Dead-sure |
| i18n under `proposals.show.header.*` with no `t(` left | Dead-sure after view delete |
| Feature CSS file whose BEM only appeared in deleted ERB | Dead-sure — delete CSS + `@import` |
| Tailwind-only deleted ERB (no custom class in stylesheets) | No CSS file to delete — note in audit |
| Houses/lands `unless modal_frame…; redirect` | Pattern to copy for modal-only |

---

## Anti-patterns

| Don’t | Why |
|-------|-----|
| Delete `show.html.erb` while non-frame GET still renders it | Runtime 500 / missing template |
| Delete shared `RelatedForCustomerQuery` because history bar died | Breaks customer proposals tab |
| “While here” prune 10 payload keys without updating service specs | Noise + risk in UI cleanup PR |
| Assume Screenshot = dead without grep | Specs/redirects/fallback may still hit it |
| Change shared helper defaults to “fix” one dead screen | Violates `shared-abstraction-safety` |

---

## Related skills / rules

- `structural-change-analysis` — before large delete/split plans (Stage 1–2)
- `shared-abstraction-safety.mdc` — blast radius on shared helpers
- `integration-regression-review` — Turbo frame / modal wiring after cutover
- `ai-housemaker-review-checklist` — P1 “Dead services” row; load this skill for UI migration deadcode
- `code-review` — attach audit table under findings when Stage 8

---

## Example (proposals show → modal)

| Item | Class | Action taken |
|------|-------|--------------|
| `_modal_property_summary`, JS `replacementMeta` | Dead-sure | Delete immediately |
| `show` + `_header` / `_media_hero` / … / history bar | Fallback-only then Dead | Modal-only redirect + delete |
| `RelatedProposalRow` + emoji helpers | Dead after bar gone | Delete with bar |
| `RelatedForCustomerQuery` | Shared-keep | Keep (customer tab) |
| `ShowPayload` `:tags` / `:features` / `:type_badge` / gallery arrays | Soft-dead payload | Follow-up prune (optional) |
