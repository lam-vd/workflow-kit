# Project profile: manet

> **Read order:** Load **only** the § section for the current stage.
>
> **EN**: Delta for `Documents/workspace/port_jp/manet` (Yenom / ma-net.jp).
> **VI**: Card_loan CMS + Super admin + Handsaw + Pundit. Không dùng ai-housemaker Hotwire hay telemedease Grape/PayJP. Pundit: local `manet-admin-cms` (không symlink ahm tenant policies).

## Detection

Load when **any** of:

1. **Auto**: cwd or staged paths contain `manet/` or `port_jp/manet` (not `manet-adwords`)
2. **Override**: `(manet)` / `/review-staged manet`

**Do not** load `ai-housemaker.md`, `telemedease.md`, or `skal.md` for this repo.

> Path note: `manet-adwords-analysis--offline-cv` is a **different** repo — do not treat as manet profile.

---

## §1 — Stage 1: `/analyze-task`

**Read first:**

1. `manet/AGENTS.md` — card_loan vs FX, soft-delete, skills routing
2. `manet/.cursor/rules/project/manet-overview.mdc`
3. `external-side-effect-confirm` — before any live Airbrake / staging-prod / destructive DB
4. `dry-duplication-scan` — if adding behavior
5. `field-impact-analysis` — if fields may change
6. Domain skill: `manet-card-loan` / `manet-admin-cms` / `manet-markdown-handsaw`

**Reflect:**

| Area | Reflect |
|------|---------|
| Surface | public card_loan / super / processors |
| FX | Must stay out unless DoD |
| Soft-delete | Which column convention? |
| Cache | Markdown / CloudFront? |
| Authz | Super `rule` + permit flags |

Flag 🟠/🔴 for Super auth, shared processors, ranking/SEO schema, cache invalidation.

---

## §3 — Stage 3: `/write-spec`

Cite real paths under `manet/`. Output: `docs/specs/<date>-<slug>.md` (EN).

Must cover when relevant: PC/SP(+AMP), soft-delete, Pundit, Handsaw cache, FX exclusion.

DRAFT until `/check-spec`. No Hotwire / teleme portal templates.

---

## §7 — Stage 7: `/start-coding`

**Read:** matching `manet-*` skill + `fx-legacy-guard` if near FX + `admin-super-isolation` / `pundit-admin` if super + `markdown-processors` if Handsaw + `dry-duplication-scan`.

**Hard rules:**

- **Confirm gate:** load `external-side-effect-confirm` before Airbrake-triggering runs, staging/prod hits, `db:drop`/`db:reset`, or real mailers — wait for explicit OK
- No Grape / PayJP / CanCanCan / Refile / Hotwire / Active Storage patterns
- Services: `SomeService.call` (`BaseService`) — not skal `execute`
- Soft-delete: match model column
- Tests: RSpec under `spec/`

```bash
docker compose exec -T app bundle exec rspec <paths>
docker compose exec -T app bundle exec rubocop --force-exclusion <paths>
```

Agent stages only; never commit/push unless user asks.

---

## §8 — Stage 8: `/review-staged`

1. `code-review`
2. `edge-case-boundary-review` — unless copy/CSS-only
3. `idor-prevention` — if super/controllers touch IDs
4. `manet-markdown-handsaw` — if processors staged
5. `manet-admin-cms` — if Super/Pundit staged
6. `feature-cutover-orphan-review` — if removing CMS UI keep domain
7. `lean-facade-review` — thin getters / I18n `default:`
8. `pr-review-comments` — if paste-ready comments requested

**Do not** load: `rails-tl-review`, Hotwire, `teleme-payment`, ahm tenant pundit.

**READY:** No 🔴/🟠; specs pass; FX untouched unless DoD.

**Commit:** print for user; agent never commits.

---

## §9b — Stage 9b: `/create-pr`

English PR body via `manet-pr` + `workflow/pr-description-quality.mdc`.

Sections: Description · Root Cause · Solution · Impact · Risk Assessment · Testing.
