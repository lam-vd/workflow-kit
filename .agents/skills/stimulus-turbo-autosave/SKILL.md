---
name: stimulus-turbo-autosave
description: "Stimulus + Turbo patterns for field/checkbox autosave (debounce, serial coalesce, option-only turbo path, silent-fail UX). Use when implementing or reviewing PATCH-on-change autosave in modals (e.g. proposal house options). Pair with ai-housemaker-rspec HARD BAN (verify DB in request, UI in Manual QA)."
---

# Stimulus + Turbo autosave

> Lessons from ai-housemaker proposal option autosave (No.130–132, Jul 2026).  
> Canonical RSpec layer rules: `ai-housemaker-rspec` / `rspec-patterns` (**no UI in request**).

## When to use

- Checkbox / toggle / field that **PATCHes immediately** on change
- Open Turbo Frame / Preline modal must **not jump scroll** on every save
- Rapid toggles risk abort storms or last-write races

## Pattern summary

| Concern | Do | Don't |
|---------|----|-------|
| Rapid input | Debounce (~300ms) then flush | Fire PATCH on every `change` with only `AbortController` |
| Concurrent PATCH | Serial coalesce — one in-flight; queue “pending”; flush latest DOM on `finally` | Parallel PATCHes; discard latest selection |
| Status/memo submit during autosave | Clear debounce; if in-flight → `preventDefault`, await promise, `requestSubmit` once with latest DOM | Abort-only (request may already be on server) or let stale PATCH finish after submit |
| Server response | Prefer `option_only` / partial streams or `204` — **skip full detail-frame replace** | Always `turbo_stream.replace` whole modal |
| Auth | CSRF meta + `credentials: "same-origin"` + `Accept: text/vnd.turbo-stream.html` | Cookie-less fetch |
| Failure UX | Toast and/or revert checkbox (follow-up if missing) | Silent `if (!response.ok) return` forever without Manual QA note |
| Tests | Request: status/`media_type`/`204` + **DB**; Manual QA: scroll, debounce, offline | `expect(response.body).to include('…option-choice…')` |

## Controller sketch (Stimulus)

```js
// On change: update UI immediately, then scheduleAutosave()
// scheduleAutosave: set pending, clearTimeout, setTimeout(flush, DEBOUNCE_MS)
// flush: if inFlight → keep pending; else PATCH FormData of current checkboxes
// finally: clear inFlight; if pending → flush again with latest DOM
```

Reference: `ai-housemaker/app/javascript/controllers/proposal_option_selection_controller.js`

## Rails sketch

```ruby
# After successful Update, when params are choices-only:
if option_only_update?(update_params)
  # history streams optional; do NOT replace detail modal frame
  return render turbo_stream: streams.presence || head(:no_content)
end
```

Detect option-only: `house_option_choice_ids` present **and** no `status` / `memo` keys.

## Hotwire pattern IDs

| ID | See |
|----|-----|
| **TURBO-AUTOSAVE-01** | Debounce + serial coalesce |
| **TURBO-AUTOSAVE-02** | Skip full-frame replace on field-only PATCH |
| **TURBO-AUTOSAVE-03** | Failure feedback (toast/revert) — P2 if silent |
| **TURBO-AUTOSAVE-04** | Await in-flight autosave before status/memo form submit (no stale overwrite) |
| **TURBO-STREAM-HOOK-01** | Turbo 8: use `turbo:before-stream-render` wrap, not `turbo:after-stream-render` |

Catalog: `hotwire-integration-patterns.mdc`

## Stage 7–8 checklist

- [ ] Debounce + coalesce implemented (or documented why not)
- [ ] Server path avoids full modal replace for field-only updates
- [ ] Request specs assert **DB + HTTP**, not markup
- [ ] Manual QA: rapid toggle, offline/422, scroll position, race with status/memo submit (toggle A → in-flight → change B → submit → DB = B)
- [ ] If silent fail remains → 🟡 finding + follow-up, not new request UI asserts

## Related

- `hotwire-integration-patterns.mdc`
- `ai-housemaker-rspec` / `rspec-patterns`
- `ai-housemaker-review-checklist`
- `shared-abstraction-safety.mdc` — do not put feature autosave into shared submit helpers
