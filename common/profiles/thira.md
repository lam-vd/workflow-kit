# Project profile: thira

> **Read order:** Load **only** the § section for the current stage.
>
> **EN**: Delta for `port_jp/thira` — Vue 2 static Manet LP / A/B.
> **VI**: Không Rails, không OCV batch, không Hotwire. Ad params (`kw_id`) là CRITICAL.

## Detection

Load when **any** of:

1. **Auto**: paths contain `thira/` or `port_jp/thira`
2. **Override**: `(thira)` / `/review-staged thira`

**Do not** load `manet.md`, `manet-adwords-ocv.md`, `telemedease.md`, `skal.md`, or ai-housemaker Hotwire.

---

## §1 — Stage 1: `/analyze-task`

**Read first:**

1. `thira/AGENTS.md`
2. `.cursor/rules/project/thira-overview.mdc`
3. `external-side-effect-confirm` — before deploy / reg-suit S3 publish / Slack notify / prod workflow_dispatch
4. `dry-duplication-scan` under `src/js/`
5. `thira-ab-landing` / `thira-kw-id-config` if obvious

**Reflect:** page · rabbits/turtles · `kw_id`/segment blast · Storybook/Cypress.

Flag 🟠/🔴 for live ad config / default card order / attribution params / unconfirmed publish.

---

## §3 — Stage 3: `/write-spec`

Cite `src/` paths. EN spec. Cover `kw_id` matrix and no-bleed. No Rails templates. DRAFT until `/check-spec`.

---

## §7 — Stage 7: `/start-coding`

**Read:** matching `thira-*` + `ad-query-params` + `vue2-static-webpack` + `dry-duplication-scan`.

**Hard rules:**

- **Confirm gate:** `yarn deploy`, reg-suit S3 publish, Slack notify, prod CI dispatch → `external-side-effect-confirm` first
- Edit `src/` only; no `dist/`
- Node 20.18.0; no Vue 3 without DoD
- Gate experiments via Util; protect unknown `kw_id`
- Update Storybook if components move
- Agent stages only; never commit/push unless asked

```bash
yarn lint && yarn test
```

---

## §8 — Stage 8: `/review-staged`

1. `code-review`
2. `edge-case-boundary-review` — empty/unknown `kw_id`, segment modal
3. `thira-kw-id-config` — if config/util staged
4. `thira-cypress-reg-suit` — if UI flows staged
5. `pr-review-comments` — if paste-ready comments requested

**Do not** load Rails/OCV/Hotwire skills.

**READY:** No 🔴/🟠; lint/test OK; no-bleed checked when ads touched.

**Commit:** print for user; agent never commits.

---

## §9b — Stage 9b: `/create-pr`

English PR via `thira-pr` + A/B checklist when relevant.
