# Project profile: evekatsu

> **Read order:** Load **only** the § section for the current stage.
>
> **EN**: Delta for `port_jp/evekatsu` — event portal, multi-dimension routes, PC/SP, Pundit admin.
> **VI**: Không Hotwire / teleme Grape. Routing + cache là CRITICAL.

## Detection

1. **Auto**: paths contain `evekatsu/` or `port_jp/evekatsu`
2. **Override**: `(evekatsu)` / `/review-staged evekatsu`

**Do not** load ai-housemaker Hotwire, telemedease, thira, or manet CMS profiles.

---

## §1 — Stage 1: `/analyze-task`

**Read:** `evekatsu/AGENTS.md` → `evekatsu-overview.mdc` → `dry-duplication-scan` → `eve-events-search` / `eve-admin` / `eve-ads-sponsors` if obvious.

**Reflect:** public vs admin · route dimensions · cache/CDN lag · Place polymorphic · PC/SP.

Flag 🟠/🔴 for routing/SEO, cache keys, Pundit, sponsor/ad placement.

---

## §3 — Stage 3: `/write-spec`

Cite real paths. EN spec. Cover route matrix, setsumeikai redirects, PC/SP, cache. DRAFT until `/check-spec`.

---

## §7 — Stage 7: `/start-coding`

**Read:** matching `eve-*` + `event-routing` / `event-cache` / `pc-sp-cloudfront` / `admin-session` as needed + `dry-duplication-scan`.

**Hard rules:**

- `SomeService.call` idiom
- Public UI: PC + `+sp`
- Do not rename `cache_clearner`
- RSpec under `spec/`
- Agent stages only; never commit/push unless asked

```bash
docker compose exec -T rails bundle exec rspec <paths>
# or local bundle exec rspec / rubocop
```

---

## §8 — Stage 8: `/review-staged`

1. `code-review`
2. `edge-case-boundary-review` — empty filters, graduation bounds, place polymorphic
3. `idor-prevention` — if admins/IDs
4. `eve-events-search` — if routing/search staged
5. `feature-cutover-orphan-review` — if removing UI keep domain
6. `pr-review-comments` — if paste-ready comments requested

**READY:** No 🔴/🟠; specs pass; PC/SP checked when views change.

**Commit:** print for user; agent never commits.

---

## §9b — Stage 9b: `/create-pr`

English PR via `eve-pr`.
