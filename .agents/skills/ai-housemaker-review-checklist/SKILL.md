---
name: ai-housemaker-review-checklist
description: "Pre-merge checklist from real ai-housemaker PR review (profile UI). Use at Stage 7–8 for Rails/Hotwire forms, modals, ActiveStorage, audit logs, Stimulus file preview, RSpec layers. Loaded via common/profiles/ai-housemaker.md at /review-staged."
---

# ai-housemaker Review Checklist

> Condensed from PR #92 profile feedback. **Implement + self-review** before `/review-staged`.

## When to use

- `app/controllers/**`, `app/services/**`, `app/forms/**`, `app/javascript/controllers/**`
- `spec/requests/**`, `spec/support/shared_examples/**`
- Feature CSS (`profile-entry.css`, `auth-entry.css`), shared `components.css`
- Turbo frames/streams, modals, file upload

Project rules: `ai-housemaker/.cursor/rules/implementation/audited-active-storage.mdc`, `quality/stimulus-file-input-preview.mdc`, `quality/rspec-best-practices.mdc`, `security/brakeman.mdc`.

RSpec skill: `.agents/skills/ai-housemaker-rspec/SKILL.md`.

---

## P0 — block merge

| Area | Check | Fail pattern |
|------|-------|--------------|
| **Audited ActiveStorage** | Avatar attach/purge + scalar fields = **one service, one transaction** | `UpdateAvatar` / `PurgeAvatar` + `Update` — orphan services or split audit |
| **File input cancel (削除)** | `deleteImage` calls `input.value = ''` **before** restore preview | UI resets but browser still submits file |
| **Brakeman** | Full scan exit 0 before push | `VerbConfusion`: `request.get?` on GET+POST action — use `request.get? \|\| request.head?` |

---

## P1 — fix in PR

| Area | Check | Fail pattern |
|------|-------|--------------|
| **Request spec scope** | `spec/requests/**` = status, redirect, flash, headers, DB, jobs — **no UI in body** | `include('プロフィール')`, `I18n.t` in body, DOM IDs, CSS class counts |
| **Functional verification** | Staged-related `rspec` passes; commands recorded in review report | Spec fail on branch feature → BLOCKED |
| **Hotwire integration** | Regression sweep per `hotwire-integration-patterns.mdc` — no P1 FAIL | Pagination wrong URL after detail; stale header count after filter |
| **Search + pagination** | Sweep `paginated-list-patterns.mdc` PAGE-* + PAGE-CTX-01 | `q`/`page` lost on modal delete; overflow without history-replace |
| **Pagination destroy** | `25da272` pattern: repage returns page → `history-replace` if corrected | Empty last page after delete; URL still `page=N` |
| **remove_avatar after 422** | Hidden field + preview from `@form.remove_avatar?`; Stimulus must not reset flag on `connect()` | Hardcoded `value: "0"`; user loses purge intent after validation error |
| **Re-open picker → Cancel** | `!file` → restore preview (browser cleared input) | Keep blob preview while input empty — submit sends no file |
| **Remove w/o attachment** | Enable 削除 when pending upload (`objectUrl` or `files.length`) | User with no avatar cannot cancel wrong file pick |
| **Blob preview `alt`** | No `alt=""` on `createObjectURL` preview; server img uses `full_name \|\| email` | Empty alt on dynamic preview |
| **Dead services** | Delete orphan services/specs after consolidating into one path | `PurgeAvatar` / `UpdateAvatar` with no callers |
| **Dead auth guard** | No `signed_in?` branch after global `authenticate_*!` unless `skip_before_action` | Unreachable error path |
| **Form normalize** | `before_validation` strip/downcase email (mirror `ProfileForm`) | Format validator runs on raw `"  A@B.com  "` |
| **Shared CSS scope** | Feature-specific cursor/layout in feature CSS | Global change to `.ahm-modal-button` in `components.css` |
| **Turbo sync** | Profile update streams replace header avatar/name partials if page shows them | Stale header after save |
| **DRY display** | Call model/class method (`TenantUser.format_phone_for_display`) — no thin helper wrapper | One-line delegate helper |

---

## P2 — should fix / QA note

| Area | Check |
|------|-------|
| **CSS comments** | Keep Figma IDs / non-obvious rules; drop echo comments (`/* 768px */`) |
| **CSS specificity** | Feature CSS after Tailwind; avoid utility classes that override Figma tokens |
| **Stale assets** | CSS/JS unchanged in browser → `yarn build` + hard refresh; CSS → `bin/rails assets:clobber` |
| **Email pending UI** | Info box = `@pending_email_change` (server, `EmailChangeRequest.active`) — **not** JS auto-hide timer |
| **Shared examples** | Turbo stream examples: `have_http_status` + `media_type` only — no partial IDs in body |

---

## Defer — do not block PR

- Split `email_change` GET/POST into separate actions (follow-up; modal pattern OK for now)
- System spec for full avatar modal / file-picker cancel UX (manual QA OK for now)
- Compensating audit rollback service without full transaction pattern

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
| Brakeman warning | 🟠 |
| UI assertions in request spec, dead services, remove_avatar lost on 422 | 🟠 |
| Wrong cancel preview behavior, missing normalize, missing turbo partial | 🟡 |
| Echo CSS comments, deferrable route split | 🟢 |
