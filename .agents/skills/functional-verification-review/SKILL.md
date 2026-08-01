---
name: functional-verification-review
description: "Stage 8 supplement — verify staged/PR-branch behavior with automated tests and action-level checks, not diff-only review. Maps spec actions to RSpec/curl/lint runs, executes them, and reports pass/fail with reproduction for failures. Use at /review-staged alongside integration-regression-review."
---

# Skill: Functional Verification Review

## Purpose

`/review-staged` must confirm **features work**, not only that code looks correct.

| Review type | Catches |
|-------------|---------|
| Diff review | Typos, architecture, security smells |
| Integration review | Cross-layer wiring (Turbo, URL, sync) |
| **Functional verification** | Broken create/update/filter, wrong status code, regression in related specs |

---

## Scope: staged vs PR branch

| Scope | Command | When |
|-------|---------|------|
| **Staged** (commit gate) | `git diff --cached` | Always — what will be committed now |
| **PR branch** (feature context) | `git diff <base>...HEAD` | Always — full feature behavior on branch |
| **Base branch** | `git merge-base HEAD main` or `origin/main` | Detect default base |

```bash
BASE=$(git merge-base HEAD origin/main 2>/dev/null || git merge-base HEAD main)
git diff --stat "$BASE"...HEAD
git log --oneline "$BASE"..HEAD
```

**Rule:** Derive **action checklist** from PR-branch diff + FINAL spec. Run tests for **both** staged paths and related branch files.

---

## Phase 1 — Action inventory

From spec DoD + staged/branch diff, list **user/system actions**:

```markdown
| Action | Layer | Staged? | Existing spec path | Verify method |
|--------|-------|---------|-------------------|---------------|
| Create house series via modal | BE+FE | yes | spec/requests/house_series_spec.rb | RSpec + manual QA |
| Search filters list | FE+BE | yes | same | RSpec GET + manual |
```

Include: happy path, validation error (422), authz denial, empty state, pagination edge.

### Search + pagination matrix (mandatory when list has `q` / filters)

| Action | Preconditions | Expected |
|--------|---------------|----------|
| Search narrows results | On `page=3`, new `q` fits 1 page | `page=1` (or clamped), filtered rows |
| Paginate while filtered | `q=foo`, multiple pages | URL has `q` + `page=N`; hrefs keep `q` |
| Delete last on filtered last page | `q=foo&page=2`, 1 row on page | Repage to last filtered page; `q` preserved; `history-replace` if page changed |
| Delete from detail modal on page 5 | URL `?page=5&q=foo` | DELETE carries or recovers `page`/`q` (referer or `window.location`) |
| GET overflow while filtered | `q=foo&page=99` | Redirect `q=foo&page=last` (**PAGE-GET-01** — verify implemented) |
| Clear search on high page | `q=foo&page=3`, clear input | Full list page 1 |

See `.cursor/rules/paginated-list-patterns.mdc` § Manual QA.

---

## Phase 2 — Automated verification (run, don't skip)

### Rails / ai-housemaker

```bash
# Map staged files → specs
git diff --cached --name-only -- '*.rb' '*.erb'
# Run related request/model/service specs (expand paths from inventory)
docker compose exec -T app bundle exec rspec spec/requests/house_series_spec.rb
# Or targeted examples:
docker compose exec -T app bundle exec rspec spec/requests/house_series_spec.rb:514

# Stimulus changed → rebuild assets before manual QA
docker compose exec -T app yarn build

# Security gate (already in ai-housemaker prompt)
docker compose exec -T app bundle exec brakeman -q --no-pager
```

### Generic backends

| Stack | Command pattern |
|-------|-----------------|
| RSpec | `bundle exec rspec <mapped_paths>` |
| Minitest | `bin/rails test <paths>` |
| pytest | `pytest <paths> -q` |
| Jest/Vitest | `yarn test <paths>` |

### HTTP smoke (when no spec exists yet)

```bash
# Example: GET index with auth cookie / token — adapt per project
curl -s -o /dev/null -w "%{http_code}" http://localhost:3000/properties/house-series
```

**Report:** command run, exit code, pass/fail count. Failures → 🔴/🟠 finding with stack trace excerpt.

---

## Phase 3 — Branch-level regression

After staged specs pass, run **wider** checks if branch diff touches shared code:

```bash
# Example: all request specs in touched domain folder
docker compose exec -T app bundle exec rspec spec/requests/house_series_spec.rb
```

If CI script exists (`bin/pre-push-check`, `bin/ci`), prefer that for branch confidence.

---

## Phase 4 — Manual QA script (mandatory for UI actions)

For each UI action **without** system spec coverage, output table:

```markdown
## Manual QA script

| # | Action | Preconditions | Steps | Expected | Result |
|---|--------|---------------|-------|----------|--------|
| 1 | Create via modal | Admin, ≥1 house | Open modal → fill → submit | Toast + list row | PASS/FAIL/NOT RUN |
| 2 | Paginate after detail | ≥11 series | Open row → page 2 | URL `/house-series?page=2` | … |
```

Agent MUST mark `NOT RUN` with reason (no server, no seed data) — never fake PASS.

**Preconditions** always include: role, data counts, starting URL.

---

## Phase 5 — Functional findings format

```markdown
### <action> fails — **Cao (P1)** 🟠 `[functional]`

- **Hiện tượng:** `rspec spec/...:514` returns 500 / wrong redirect
- **Likelihood:** Cao — every modal create
- **Ảnh hưởng:** User cannot create series
- **Command:** `bundle exec rspec spec/requests/house_series_spec.rb:514`
- **Output:** <excerpt>
- **Cách tái hiện:** <steps or use Manual QA row #N>
- **Gợi ý:** <fix direction>
```

Tag: `[functional]`, `[regression]`, `[test-gap]`.

---

## Verdict interaction

| Situation | Verdict impact |
|-----------|----------------|
| Staged-related spec fails | ❌ BLOCKED (🟠 minimum) |
| Branch spec fails outside staged files | 🟠 BLOCKED if same feature domain |
| No spec + Manual QA FAIL | ❌ BLOCKED |
| No spec + Manual QA NOT RUN (UI change) | 🟡 flag `[test-gap]` — user must run QA or approve defer |
| Only 🟢 nits + all automated green | Can be READY |

---

## Test gap policy

UI/Turbo bugs often need manual QA. When deferring system spec:

1. Manual QA script row must be **executable** (not vague).
2. PR description How to test must copy the script.
3. Flag `[test-gap]` in review report — not hidden.

---

## ai-housemaker notes

- **HARD BAN:** Request specs assert status, redirect, flash hash, DB, headers, `media_type` — **not** body copy/DOM/CSS/Stimulus (see `ai-housemaker-rspec` / `rspec-patterns`).
- Functional verification for UI copy/layout/hover/scroll → **Manual QA script**, not new request spec assertions.
- Autosave / replace-button enable → DB request specs + Manual QA (`stimulus-turbo-autosave`, TURBO-AUTOSAVE-*, TURBO-STREAM-HOOK-01).
- After Stimulus/CSS change: `yarn build` + note stale assets (`assets:clobber` if needed).
