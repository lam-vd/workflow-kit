---
name: integration-regression-review
description: "Stage 8 supplement — trace cross-layer bugs beyond the staged diff. Maps Turbo/Hotwire/SPA flows, URL/history state, frame boundaries, i18n templates used by Stimulus, and CSS calc/line-height mismatches. Outputs regression sweep table + findings with data/URL preconditions and reproduction steps. Use with functional-verification-review at /review-staged."
---

# Skill: Integration & Regression Review

## When to use

Run at **Stage 8** (`/review-staged`) when staged files include any of:

- `app/views/**`, `app/javascript/**`, `*.css` / `*.scss`
- `config/locales/**`
- Controllers with `turbo_stream`, `respond_to`, redirects
- API clients, axios services, Zustand stores (FE apps)

**Also read** when reviewing fix commits: compare `git diff <parent>..HEAD` to map each hunk → symptom.

**Project-specific patterns:**
- ai-housemaker Hotwire → `.cursor/rules/hotwire-integration-patterns.mdc`
- Paginated list + search/filter → `.cursor/rules/paginated-list-patterns.mdc`

---

## Mindset

Static diff review catches **syntax and local logic**. Integration review catches **wiring bugs**:

- Element updated inside turbo frame but counter/header **outside** frame stays stale
- `turbo_action: advance` + modal `Turbo.visit` → pagination links keep **member `:id`** in URL
- POST from modal → browser URL stuck on `/new` until refresh breaks
- Locale `__COUNT__` template typo → Stimulus replace shows wrong JA copy
- CSS `min-height` uses variable A but `line-height` uses variable B → clipping
- Search on page 5 narrows results but URL keeps `page=5` → empty filtered list
- Pagination links drop `q` after `paginate(..., params: { controller:, id: nil })` only

---

## Phase A — Diff → symptom hypothesis

For each meaningful hunk in staged diff, fill one row:

| Change (file) | Hypothesized symptom | Who sees it | Read next (1-hop) |
|---------------|---------------------|-------------|-------------------|
| `pagination_params` in partial | Page 2 links to show URL | User paginating after detail | `row_link_controller`, routes |
| `total-count-sync` controller | — (fix) | — | `index.html.erb` header id |

**Rule:** If you cannot name a user-visible symptom, either drop the row or flag 🟡 "unverified change".

---

## Phase B — Integration flow trace (mandatory)

For each **user-facing action** touched by the diff, trace:

```
User gesture
  → Stimulus / link handler
  → Turbo visit | form submit | frame load
  → Controller#action
  → turbo_stream targets | redirect | JSON response
  → DOM outside frame updated?
  → Browser URL (pushState / replaceState)?
```

**Commands (adapt to repo):**

```bash
# Turbo targets in staged views
git diff --cached -- '*.erb' '*.html' | rg 'turbo_frame_tag|turbo_stream'

# Stimulus controllers touched
git diff --cached --name-only -- '**/controllers/*.js'

# Who calls a new controller (1-hop callers)
rg '<controller-name>' app/views app/javascript --glob '*.{erb,js}'

# Locale keys used by Stimulus templates
rg '__COUNT__|_template_value|I18n' app/views config/locales
```

**Output:** one bullet flow per primary action (create, update, filter, paginate, open modal).

---

## Phase C — Regression pattern sweep

Load project pattern rule if present (e.g. `hotwire-integration-patterns.mdc`). For each pattern row:

| Pattern ID | Checked | Result | Evidence |
|------------|---------|--------|----------|
| TURBO-SYNC-01 | ✅ | PASS / FAIL / N/A | … |

- **FAIL** on P0/P1 pattern → finding with reproduction (see Phase D).
- **N/A** only when diff cannot trigger pattern (document why).

### Phase C2 — Paginated list + search (when listing/filter/pagination staged)

Load `.cursor/rules/paginated-list-patterns.mdc`. Sweep at minimum:

| Pattern ID | Checked | Result |
|------------|---------|--------|
| PAGE-OVERFLOW-01 | ✅ | PASS / FAIL / N/A |
| PAGE-GET-01 | ✅ | … |
| PAGE-SEARCH-01 | ✅ | … |
| PAGE-SEARCH-02 | ✅ | … |
| PAGE-SEARCH-04 | ✅ | … |
| PAGE-SEARCH-07 | ✅ | … |
| PAGE-CTX-01 | ✅ | … |
| PAGE-COERCE-01 | ✅ | … |

Trace **single source of truth** for index params: same helper for paginate, redirect, delete URL, history-replace?

```bash
rg 'paginate|pagination_nav|fetch_page|params\[:page\]|params\[:q\]' app/controllers app/views app/helpers --glob '*.{rb,erb}'
rg 'auto-submit|turbo_frame.*list' app/views app/javascript
```

When no project rule exists, run generic checks:

| ID | Check |
|----|-------|
| `INT-URL-01` | Pagination/filter links preserve wrong path params after nested navigation |
| `INT-SYNC-01` | Partial page update leaves related UI outside update boundary stale |
| `INT-HIST-01` | Modal/stream success leaves browser URL not restorable on refresh |
| `INT-I18N-01` | Dynamic label uses template key inconsistent with static copy |
| `INT-CSS-01` | `line-height` / `min-height` / `padding-block` use mismatched tokens |

---

## Phase D — Reproduction spec (P0/P1 mandatory)

Every 🔴/🟠 finding MUST include:

```markdown
### <title> — P<n>

- **Hiện tượng:** <file/behavior>
- **Likelihood:** Cao | Trung bình | Thấp | Có điều kiện — <trigger>
- **Ảnh hưởng:** <user-visible outcome>
- **Điều kiện data:** <record counts, roles, seed state>
- **Điều kiện URL / state:** <path, query, open modal, frame id>
- **Cách tái hiện:**
  1. …
  2. …
- **Quan sát (bug):** …
- **Verify nhanh:** DevTools Network / address bar / `#element-id`
- **Gợi ý / Fix:** …
```

Invalid finding if P0/P1 lacks **Điều kiện data** or **Cách tái hiện**.

---

## Phase E — Fix-commit coverage (when sub-task is a bugfix)

```bash
git diff <parent-commit>..HEAD --stat
git diff <parent-commit>..HEAD
```

Output **Fix coverage table:**

| Symptom | Root cause | Fixed by (file) | Repro verified? | Test added? |
|---------|------------|-----------------|-----------------|-------------|
| Header count stale after search | Counter outside turbo frame | `total_count_sync_controller.js` | Manual / RSpec | … |

---

## Severity hints

| Pattern | Typical severity |
|---------|------------------|
| Wrong URL on pagination after detail | 🟠 P1 |
| Stale total count after filter | 🟠 P1 |
| Page overflow after delete (empty list) | 🟠 P1 |
| Search + high page → empty list (PAGE-SEARCH-01) | 🟠 P1 |
| Pagination drops `q` / filter params (PAGE-SEARCH-02) | 🟠 P1 |
| Delete/repage drops filter params (PAGE-SEARCH-04/07) | 🟠 P1 |
| URL broken on refresh after modal create | 🟠 P1 |
| JA copy typo in dynamic label | 🟡 P2 |
| CSS vertical clip in table/detail | 🟡 P2 |

---

## Anti-patterns (reviewer)

- Reviewing only `git diff --cached` without reading 1-hop neighbors
- Finding without user-visible symptom
- "Test manually" without concrete steps and preconditions
- Blocking on 🟢 CSS nits while missing 🟠 Turbo URL bug
