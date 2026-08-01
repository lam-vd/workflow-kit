---
name: ai-housemaker-review-checklist
description: "Pre-merge checklist from real ai-housemaker PR reviews (profile UI, nested Preline/Turbo modals, Hotwire). Use at Stage 7–8 for Rails/Hotwire forms, modals, ActiveStorage, audit logs, Stimulus, RSpec layers. Loaded via common/profiles/ai-housemaker.md at /review-staged."
---

# ai-housemaker Review Checklist

> Condensed from PR #92 profile feedback **+** proposal nested-memo / Hotwire incidents **+** proposal viewed lifecycle / option autosave (Jul 2026) **+** detail modal canonical URLs No.139 (Jul–Aug 2026). **Implement + self-review** before `/review-staged`.

## When to use

- `app/controllers/**`, `app/services/**`, `app/forms/**`, `app/javascript/controllers/**`
- `spec/requests/**`, `spec/support/shared_examples/**`
- Feature CSS (`profile-entry.css`, `auth-entry.css`, `proposal-memo.css`, …), shared `components.css`
- Turbo frames/streams, **nested Preline modals**, file upload

Project rules: `ai-housemaker/.cursor/rules/implementation/audited-active-storage.mdc`, `quality/stimulus-file-input-preview.mdc`, `quality/hotwire-integration-patterns.mdc`, `quality/rspec-best-practices.mdc`, `security/brakeman.mdc`.

RSpec skill: `.agents/skills/ai-housemaker-rspec/SKILL.md` (canonical detail: `ai-housemaker/.agents/skills/rspec-patterns/SKILL.md`).  
Autosave: `.agents/skills/stimulus-turbo-autosave/SKILL.md`.  
Canonical detail modal URLs: `.agents/skills/modal-detail-canonical-url/SKILL.md`.

---

## P0 — block merge

| Area | Check | Fail pattern |
|------|-------|--------------|
| **Audited ActiveStorage** | Avatar attach/purge + scalar fields = **one service, one transaction** | `UpdateAvatar` / `PurgeAvatar` + `Update` — orphan services or split audit |
| **File input cancel (削除)** | `deleteImage` calls `input.value = ''` **before** restore preview | UI resets but browser still submits file |
| **Brakeman** | Full scan exit 0 before push | `VerbConfusion`: `request.get?` on GET+POST action — use `request.get? \|\| request.head?` |
| **Nested Preline + teleport** | Child overlay teleported to `document.body`; parent-frame **host** removes it on disconnect | Child stuck under parent dimmer (PRELINE-NEST-01) |

---

## P1 — fix in PR

| Area | Check | Fail pattern |
|------|-------|--------------|
| **Request spec HARD BAN** | `spec/requests/**` = status, redirect, flash, headers, DB, jobs, `media_type` — **zero UI in body** | `include('閲覧済')`, `I18n.t` in body, BEM/CSS, Stimulus `data-*`, `.scan` counts, option-card markup → **🟠** |
| **Functional verification** | Staged-related `rspec` passes; commands recorded in review report | Spec fail on branch feature → BLOCKED |
| **Hotwire integration** | Regression sweep per `hotwire-integration-patterns.mdc` — no P1 FAIL | Pagination wrong URL after detail; stale header count after filter |
| **Turbo 8 stream hook** | Post-stream UI re-wire uses `turbo:before-stream-render` wrap (chain render) | Relies on `turbo:after-stream-render` (TURBO-STREAM-HOOK-01) |
| **Autosave PATCH** | Debounce + serial coalesce; field-only path skips full detail-frame replace | Abort-only storms; scroll jump every toggle (TURBO-AUTOSAVE-01/02) |
| **Search + pagination** | Sweep `paginated-list-patterns.mdc` PAGE-* + PAGE-CTX-01 | `q`/`page` lost on modal delete; overflow without history-replace |
| **Pagination destroy** | `25da272` pattern: repage returns page → `history-replace` if corrected | Empty last page after delete; URL still `page=N` |
| **422 nested modal target** | Validation fail stream replaces **teleported modal id**, not whole parent detail frame | Host disconnect wipes modal; 422 with no error UI (TURBO-MODAL-01) |
| **Preline backdrop cleanup** | Feature backdrop class stripped before `HSOverlay.open` and on host/modal `disconnect` | Screen darkens each save/fail; success toast behind black (PRELINE-BACKDROP-01) |
| **Stimulus Boolean values** | `data-*-value` always `'true'`/`'false'` (never `<%= nil %>` → `""`) | Empty open-value auto-opens modal on every parent open (STIMULUS-BOOL-01) |
| **remove_avatar after 422** | Hidden field + preview from `@form.remove_avatar?`; Stimulus must not reset flag on `connect()` | Hardcoded `value: "0"`; user loses purge intent after validation error |
| **Re-open picker → Cancel** | `!file` → restore preview (browser cleared input) | Keep blob preview while input empty — submit sends no file |
| **Remove w/o attachment** | Enable 削除 when pending upload (`objectUrl` or `files.length`) | User with no avatar cannot cancel wrong file pick |
| **Blob preview `alt`** | No `alt=""` on `createObjectURL` preview; server img uses `full_name \|\| email` | Empty alt on dynamic preview |
| **Dead services** | Delete orphan services/specs after consolidating into one path | `PurgeAvatar` / `UpdateAvatar` with no callers |
| **Dead UI after migration** | After full-page→modal (or equivalent), run `deadcode-ui-migration-review`: grep call sites; classify Dead-sure / Fallback-only / Soft-dead payload / Shared-keep; modal-only gate + redirect before deleting show stack | Delete `show` partials while non-frame GET still renders them; delete shared query used by another tab |
| **Orphan CSS after view delete** | After deleting ERB, grep BEM/custom classes in `app/assets/stylesheets`; remove feature CSS + `@import` or document Tailwind-only | Left `proposals-show.css` / unused BEM after HTML gone; deleted modal CSS still used by `_detail_modal_frame` |
| **Dead auth guard** | No `signed_in?` branch after global `authenticate_*!` unless `skip_before_action` | Unreachable error path |
| **Form normalize** | `before_validation` strip/downcase email (mirror `ProfileForm`) | Format validator runs on raw `"  A@B.com  "` |
| **Shared CSS scope** | Feature hover/cursor: call-site or additive shared cursor; **no** no-op `:hover` overrides | Feature sets hover = default colors (CSS-BTN-HOVER-01); silent global layout in `components.css` |
| **Turbo sync** | Profile / list row / history panel streams replace if page shows them | Stale header / list after modal save |
| **Live editable history** | In-meeting history detail keeps replace/options via `live_context` (or DDD equiv) | History open loses editable UI after stream (TURBO-LIVE-CTX-01) |
| **Canonical detail modal URL** | Lean load on full-page deep-link; heavy includes only on detail Turbo-Frame; capture detail record before `load_index_collection` if ivar shared; pass `filter_params` into every row/card; shell/row `data-*-url-value` asserted exactly | Bullet unused includes; garbage `detail_url`; close modal drops filters (TURBO-MODAL-URL-01..03) — see `modal-detail-canonical-url` skill |
| **Nullable enum / blank status** | `allow_nil`, blank → `nil` in params/services; UI/filter/label cover unselected | Forced `candidate` fallback; memo gated on “save status first” |
| **DRY display** | Call model/class method (`TenantUser.format_phone_for_display`) — no thin helper wrapper | One-line delegate helper |

---

## P2 — should fix / QA note

| Area | Check |
|------|-------|
| **Autosave failure UX** | Silent `!response.ok` → toast/revert or explicit Manual QA defer (TURBO-AUTOSAVE-03) |
| **CSS overflow wrap** | Value cells + ancestors: `min-width: 0` + `overflow-wrap` / `word-break` (CSS-OVERFLOW-01). Label–value grids use `minmax(0, …)`. Chip/tag groups wrap (`flex-wrap`); long chips get `max-width: 100%`. Rebuild compiled CSS after source edits. |
| **maxlength vs server length** | `maxlength` is UX; model `maximum` is authority — note TURBO-VALID-01 when QA’ing 422 UI |
| **Success UX after nested save** | Close child modal + clean backdrops; optional return to relevant tab (e.g. memo) |
| **CSS comments** | Keep Figma IDs / non-obvious rules; drop echo comments (`/* 768px */`) |
| **CSS specificity** | Feature CSS after Tailwind; avoid utility classes that override Figma tokens |
| **Stale assets** | CSS/JS unchanged in browser → `yarn build` + hard refresh; CSS → `bin/rails assets:clobber` |
| **Email pending UI** | Info box = `@pending_email_change` (server, `EmailChangeRequest.active`) — **not** JS auto-hide timer |
| **Shared examples** | Turbo stream examples: `have_http_status` + `media_type` only — no partial IDs in body |
| **Ensure-on-open GET** | When DDD mandates ensure on show: document CSRF/GET tradeoff in PR; do not “fix” by asserting UI labels in request specs |

---

## Defer — do not block PR

- Split `email_change` GET/POST into separate actions (follow-up; modal pattern OK for now)
- System spec for full avatar modal / file-picker cancel UX (manual QA OK for now)
- Compensating audit rollback service without full transaction pattern
- System spec for nested Preline backdrop stacking (manual QA + pattern IDs OK for now)

---

## Quick commands (Docker)

```bash
BASE=$(git merge-base HEAD origin/main 2>/dev/null || git merge-base HEAD main)
git diff --stat "$BASE"...HEAD
git diff --staged --name-only

docker compose exec -T app bundle exec rubocop --force-exclusion $(git diff --staged --name-only -- '*.rb')
git diff --staged --name-only -- '*.html.erb' | xargs -r docker compose exec -T app bundle exec erb_lint
docker compose exec -T app bundle exec brakeman -q --no-pager
docker compose exec -T app bundle exec rspec <related_specs>   # functional-verification-review
docker compose exec -T app yarn build   # if Stimulus changed
```

---

## Severity map (for `/review-staged` report)

| Finding | Severity |
|---------|----------|
| Orphan audit / wrong attachment on failed update | 🔴 |
| File input not cleared on explicit 削除 | 🔴 |
| Nested modal unusable under parent backdrop | 🔴→🟠 |
| Brakeman warning | 🟠 |
| 422 replaces wrong target / backdrop stack / Stimulus bool auto-open | 🟠 |
| UI assertions in request spec (**HARD BAN**), dead services, remove_avatar lost on 422 | 🟠 |
| Spec `include` hardcodes setup literals (`'alt="ao@..."'`) instead of `%(alt="#{record.email})")` | 🟡 |
| Wrong cancel preview behavior, missing normalize, missing turbo partial | 🟡 |
| Silent autosave failure UX, memo overflow wrap, maxlength vs server QA note | 🟡 |
| Echo CSS comments, deferrable route split | 🟢 |

---

## Incident log (add rows; keep short)

| When | Symptom | Pattern IDs | Lesson |
|------|---------|-------------|--------|
| Profile PR #92 | Avatar / 422 / request-spec scope | (checklist P0–P1) | One transaction; **HARD BAN** UI asserts in request specs |
| Proposal memo Jul 2026 | Auto-open memo; 422 no error UI; screen darkens; layout blowout | STIMULUS-BOOL-01, TURBO-MODAL-01, PRELINE-BACKDROP-01, CSS-OVERFLOW-01 | Teleport + replace modal id + strip backdrops + explicit bool strings |
| Proposal viewed + options Jul 2026 | Replace stuck after stream; option toggle scroll jump; hover no-op | TURBO-STREAM-HOOK-01, TURBO-AUTOSAVE-01/02, TURBO-LIVE-CTX-01, CSS-BTN-HOVER-01 | `before-stream-render` wrap; debounce+coalesce; option-only no frame replace; `live_context`; real hover tokens — verify UI via Manual QA not request body |
| Detail modal URL No.139 Jul–Aug 2026 | Bullet unused includes on paste URL; house-series modal never opens; close loses filters on houses/lands | TURBO-MODAL-URL-01..06 | Lean load on canonical; capture detail before collection overwrite; pass `filter_params` to rows; assert exact `data-*-url-value` |
