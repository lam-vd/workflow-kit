---
name: ai-housemaker-rspec
description: RSpec layer boundaries + assertion style for ai-housemaker — HARD BAN UI in request specs, interpolate subject attrs with %(), modal/proposal specs. Canonical detail in ai-housemaker/.agents/skills/rspec-patterns/SKILL.md.
---

# ai-housemaker RSpec

**Canonical (full):** `ai-housemaker/.agents/skills/rspec-patterns/SKILL.md`  
**Rule (globs `spec/**`):** `ai-housemaker/.cursor/rules/quality/rspec-best-practices.mdc`

Use this skill at **Stage 7–8** when writing or reviewing specs in ai-housemaker.

## HARD BAN — `type: :request` ≠ UI test

> **Cấm kỵ tuyệt đối:** Do **not** assert UI markup, JA copy, BEM/CSS, Stimulus wiring, or layout in `spec/requests/**`.  
> Stage 8: any violation → **🟠 BLOCKED** (`ai-housemaker-review-checklist`, `rails-tl-review`).

| ✅ Request spec | ❌ `response.body` (forbidden) |
|-----------------|-------------------------------|
| `have_http_status`, `media_type`, `redirect_to` | `I18n.t(...)` / JA labels (`閲覧済`, button text) |
| `flash[:notice]` / `flash[:alert]` (hash) | BEM classes / `.scan('class').size` |
| `*Controller::*_FRAME_ID` (Turbo routing) | `data-controller` / `data-action` / `data-*-value` |
| Nested form **param names** | Upload hints, option cards, hover/cursor |
| DB / `have_enqueued_job` / JSON | Active Storage URLs in HTML |

**Business rules** (e.g. `House::CATEGORY_IMAGE_LIMITS`, ensure no-clobber) → **model/service** specs.  
**UI DoD** (hover, scroll, replace-button enable) → **Manual QA** / system spec — never request body.

```ruby
# ❌ CẤM
expect(response.body).to include('閲覧済')
expect(response.body).to include('meeting-proposal-action--primary')

# ✅
expect(response).to have_http_status(:ok)
expect(proposal.reload.status).to be_nil
```

Canonical tables + proposal examples: **HARD BAN** + **Proposal / meeting modal** in `rspec-patterns/SKILL.md`.

## Assertion style (senior)

Bind expects to the **setup object** — do not hardcode the same email/name/id again.

```ruby
# ❌
expect(html).to include('alt="ao@example.com"')

# ✅
expect(html).to include(%(alt="#{tenant_user.email}"))
expect(response.body).to include(%(action="#{proposal_path(proposal)}"))
expect(response.body).to include(%(value="#{customer.id}"))
```

| Prefer | Avoid |
|--------|--------|
| `#{record.attr}`, path helpers, controller frame constants | Magic strings mirroring `create`/`build` |
| `%()` for HTML attribute fragments with `"` | Awkward `'alt="..."'` duplication |
| `I18n.t` for messages (service/helper) | Hardcoded JA copy in expects |

Hardcoded OK for stable tokens (BEM/`data-*-target` in **helper** specs, `aria-*`, form param names) or true derived outputs (initials).

## Modal / Turbo smoke (houses / lands / proposals)

```ruby
get edit_house_path(house), headers: { 'Turbo-Frame' => HousesController::EDIT_MODAL_FRAME_ID }
expect(response).to have_http_status(:ok)
expect(response.body).to include(HousesController::EDIT_MODAL_FRAME_ID)
expect(response.body).to include('house[floor_plan_images_attributes]')
```

Do **not** assert initial-slot CSS, Stimulus `max-images`, option-card markup, or `I18n.t('…')` in body.

Option autosave / ensure-on-open: assert **status + media_type/204 + DB** only.

## Stage 8 gate

- [ ] **HARD BAN** — no UI assertions in staged `spec/requests/**` diff
- [ ] Image limit / domain rules have model or service coverage
- [ ] Helper/HTML `include` uses `%(..."#{record.attr}"...)` — not duplicated setup literals
- [ ] UI polish (cursor/hover) covered by Manual QA, not request expects
- [ ] `docker compose exec -e RAILS_ENV=test app bundle exec rspec <related>`

## Related

- `ai-housemaker/.agents/skills/rspec-patterns/SKILL.md` (canonical)
- `ai-housemaker/.cursor/rules/quality/rspec-best-practices.mdc`
- `stimulus-turbo-autosave` — autosave debounce/coalesce; verify via DB request specs + Manual QA
- `functional-verification-review` — run specs; **do not** add UI checks to request layer
- `ai-housemaker-review-checklist` — UI-in-request = 🟠
