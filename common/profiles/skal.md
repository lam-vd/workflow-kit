# Project profile: skal

> **Read order:** Load **only** the § section for the current stage.
>
> **EN**: Delta rules for `Documents/workspace/port_jp/skal`.
> **VI**: Quy tắc bổ sung cho Skal — CMS career, ads CTR, Handsaw, Pundit, Active Storage, Minitest. Không dùng profile ai-housemaker (Hotwire) hay telemedease (Grape/PayJP).

## Detection

Load this profile when **any** of:

1. **Auto**: cwd or staged paths contain `skal/` or `port_jp/skal`
2. **Override**: user writes `(skal)` or `/review-staged skal` (etc.)

**Do not** load `common/profiles/ai-housemaker.md` or `telemedease.md` for this repo.

---

## §1 — Stage 1: `/analyze-task`

**Read first (in order):**

1. `skal/AGENTS.md` — layers, ads CTR, AASM, skills routing
2. `skal/.cursor/rules/project/skal-overview.mdc`
3. `.agents/skills/dry-duplication-scan/SKILL.md` — if task adds behavior
4. `.agents/skills/field-impact-analysis/SKILL.md` — if fields may change
5. Domain skill if obvious: `skal-ads-ab` / `skal-articles-markdown` / `skal-landing-cp-signup` / `skal-admin`

**Reflect in Stage 1 output:**

| Area | Reflect |
|------|---------|
| Surface | public CMS / ads / LP / admin |
| Ads CTR | Must use `/ads/:id`? |
| Markdown | Which filters / shortcodes? |
| Authz | Pundit role blast radius |
| CP API | External Faraday services |

**Extra hard rules:** Flag 🟠/🔴 for ad redirect/tracking, OAuth domain, shared processors, Active Storage delete orphans.

---

## §3 — Stage 3: `/write-spec`

**Preconditions:** Cite real paths under `skal/` (processors, ads, admin, external services).

**Output (default):**

| File | Language |
|------|----------|
| `docs/specs/<YYYY-MM-DD>-<kebab-slug>.md` | EN canonical (+ VI mirror OK) |

No Hotwire / Preline / telemedease portal templates.

**Must cover when relevant:** ad CTR path, article AASM, PC/SP cells, CP signup, Pundit roles.

**Hard rules:** DRAFT until `/check-spec`.

---

## §7 — Stage 7: `/start-coding`

**Read first (add to base list):**

1. Matching `skal-*` skill
2. `implementation/ads-lp.mdc` — if ads/LP
3. `implementation/markdown-filters.mdc` — if Handsaw
4. `security/admin-oauth-domain.mdc` + `pundit-admin.mdc` — if admin
5. `implementation/active-storage.mdc` — if uploads
6. `dry-duplication-scan` — Reuse Scan before each new symbol

**Extra hard rules:**

- No Grape / PayJP / CanCanCan / Refile / Hotwire patterns
- Services: `SomeService.new(...).execute` only
- Tests: **Minitest** under `test/` (not RSpec)
- Lint/test:

```bash
docker compose exec -T web bundle exec rails test <paths>
# or local bundle exec rails test / rubocop / yarn lint
```

- Agent **stages only** (`git add`); never `git commit` / `git push` unless user explicitly asks

---

## §8 — Stage 8: `/review-staged`

**Skills (when profile active):**

1. `code-review`
2. `edge-case-boundary-review` — unless copy/CSS-only
3. `idor-prevention` — if admin/controllers touch IDs
4. `skal-ads-ab` checklist — if ads staged
5. `skal-articles-markdown` — if processors staged
6. `feature-cutover-orphan-review` — if removing ad types / filters keep domain
7. `lean-facade-review` — if thin getters / I18n `default:`
8. `pr-review-comments` — if user wants paste-ready comments

**Do not** load: `rails-tl-review`, Hotwire skills, `teleme-payment`.

**READY:** No 🔴/🟠; related tests pass; ads CTR / OAuth checks OK when relevant.

**Commit:** On READY → print copy-ready `git commit` for **user**; agent never commits.

---

## §9b — Stage 9b: `/create-pr`

**English PR body** (not JA ai-housemaker format).

**Read:** `skal/.agents/skills/skal-pr/SKILL.md` + `workflow/pr-description-quality.mdc`

Required sections: Description · Root Cause · Solution · Impact · Risk Assessment · Testing.

Title + commits: English Conventional Commits.
