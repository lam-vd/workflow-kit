---
name: modal-detail-canonical-url
description: "Implement/review canonical URLs for list→detail Preline/Turbo modals (deep-link paste/refresh, history.replaceState, filter restore on close). Lessons from ai-housemaker No.139 (Jul–Aug 2026). Use at Stage 7–8 when adding member URLs for detail modals or fixing Bullet unused includes / lost filters after close."
---

# Modal detail canonical URL (list ↔ modal)

> Lessons from ai-housemaker **No.139** — 詳細モーダルを固有のURLで開けるようにする.  
> Pair with: `shared-abstraction-safety.mdc`, `hotwire-integration-patterns.mdc` (TURBO-MODAL-URL-*), `paginated-list-patterns.mdc`.

## When to use

- List row opens a **detail modal** (Turbo Frame + Preline overlay) and product wants address bar `/{index}/:id`
- Paste/refresh that URL must show **index shell + auto-open modal**
- Close modal must restore **index URL with current filters/page**
- Fixing Bullet `AVOID eager loading` on canonical full-page GET
- Fixing “close modal loses `q`/filters” after search

## Do / Don't

| Do | Don't |
|----|-------|
| Opt-in `row_link` history (`historyUrl` / `indexUrl`) — default unchanged for other lists | Put history side effects into shared `row_link` for every consumer |
| `history.replaceState` after frame load (open) and on `close.hs.overlay` (close) | `turbo_action: advance` on the **detail** frame (TURBO-URL-01) |
| Lean `find` + authorize on canonical full-page; heavy `.includes` **only** on Turbo-Frame detail request | Reuse detail-modal `includes` on the index-shell render (Bullet unused eager-load) |
| Capture detail record **before** `load_index_collection` if that method overwrites the same ivar | `detail_url: foo_path(@house_series)` after `@house_series = page` (URL becomes Relation garbage) |
| Pass **every** list filter local into row/card (`filter_params` / `index_query` / …) | Assume row partial defaults `filter_params: {}` — close restores bare index path |
| `tag.attributes(data: { … })` for Stimulus values | Flat hash without `data:` → attributes render without `data-*` prefix |
| Cross-controller `render 'other/index'` → prepend lookup prefix for relative partials | Relative `render "tabs"` from `LandsController` resolving under `lands/` |
| Spec assert **specific** shell/row data attrs (`data-modal-detail-url-detail-url-value=…`) | `include(member_path(record))` alone — list rows already contain that path (false PASS) |

## Architecture (minimal)

```
Full-page GET /index/:id (canonical)
  → authorize + lean load record
  → load index collection
  → assign deep-link (detail_url, index_url, open_on_connect: true)
  → render index + shell

Turbo-Frame GET same URL
  → heavy includes if needed
  → render detail modal frame only

Row click
  → Turbo.visit(detail_url, { frame })
  → replaceState(detail_url)
  → updateShellIndexUrl(index_url with filters)

Modal close (close.hs.overlay)
  → replaceState(shell index_url)
```

Reuse in-repo primitives when present: `ModalDetailDeepLink`, `modal_detail_row_link_data`, `modal_detail_url` Stimulus, `_modal_detail_url_shell`.

## Checklist before coding (Stage 7)

- [ ] Canonical route alias keeps **legacy** member routes for other consumers
- [ ] `row_link` history is **opt-in** (shared-abstraction-safety)
- [ ] Detail frame has **no** `turbo_action: advance`
- [ ] Canonical branch: lean load; frame branch: full includes
- [ ] Ivar for detail record ≠ collection ivar after `load_index_collection` (or capture before)
- [ ] List desktop **and** mobile pass filter locals into every row/card
- [ ] Shell `index_url` + row `indexUrl` include the same filter keys the filter form uses
- [ ] Cross-controller index render uses lookup prefix if partials are relative
- [ ] Stimulus bools are `'true'`/`'false'` strings (STIMULUS-BOOL-01)
- [ ] Cold open: tolerate Preline not ready yet (retry open) without breaking row-click path
- [ ] Request specs: deep-link marker / **exact** `data-*-url-value` path — not full modal HTML (HARD BAN UI copy)

## Hotwire pattern IDs (catalog)

| ID | Failure |
|----|---------|
| **TURBO-MODAL-URL-01** | Canonical full-page uses detail `.includes` → Bullet unused eager-load alerts |
| **TURBO-MODAL-URL-02** | `load_index_collection` overwrites detail ivar → garbage `detail_url` → paste URL never opens modal |
| **TURBO-MODAL-URL-03** | Row `indexUrl` missing filters → close modal drops `q`/status/page from address bar |
| **TURBO-MODAL-URL-04** | Spec only `include(member_path)` → false green while shell `detail-url` is wrong |
| **TURBO-MODAL-URL-05** | Cross-controller index render → `Missing partial X` under wrong prefix |
| **TURBO-MODAL-URL-06** | `tag.attributes` without nested `data:` → Stimulus never connects |

Full catalog rows: `hotwire-integration-patterns.mdc`.

## Manual QA (minimum)

1. Filter/search list → open detail → URL is `…/:id` → close → URL has filters again  
2. Paste `…/:id` in new tab → index + modal open (no Bullet alert storm)  
3. Paginate after close → URL stays index (no stuck `:id`) — TURBO-URL-01  
4. Non-opt-in lists (e.g. customers) unchanged  

## Related

- `shared-abstraction-safety.mdc` — opt-in on shared `row_link`
- `hotwire-integration-patterns.mdc` — TURBO-URL-01 + TURBO-MODAL-URL-*
- `ai-housemaker-review-checklist` — P1 rows for canonical modal URL
- `paginated-list-patterns.mdc` — PAGE-* when filters + page interact
- In-app: `ModalDetailDeepLink`, `modal_detail_url_controller.js`, `row_link_controller.js`
