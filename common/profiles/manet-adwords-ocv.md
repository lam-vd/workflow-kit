# Project profile: manet-adwords-ocv

> **Read order:** Load **only** the § section for the current stage.
>
> **EN**: Delta for `port_jp/manet-adwords-analysis--offline-cv` (batch OCV).
> **VI**: Batch Ruby / Docker build-time jobs / Sheets / ASP crawlers. **Không** dùng profile `manet.md` (CMS), telemedease, skal, hay ai-housemaker Hotwire.

## Detection

Load when **any** of:

1. **Auto**: paths contain `manet-adwords` **or** `manet-adwords-analysis--offline-cv` **or** `offline-cv` (as this repo folder)
2. **Override**: `(manet-adwords-ocv)` / `(ocv)` / `/review-staged ocv`

**Do not** load `common/profiles/manet.md` for this repo (CMS Yenom is separate).

---

## §1 — Stage 1: `/analyze-task`

**Read first:**

1. `manet-adwords-analysis--offline-cv/AGENTS.md`
2. `.cursor/rules/project/ocv-overview.mdc`
3. `external-side-effect-confirm` — **always** before `docker build` / ASP crawl / Ads·Sheets write / Slack·GChat·Airbrake / `gh workflow run`
4. `dry-duplication-scan` — search **within the target module**
5. Domain skill: `ocv-affiliate-crawler` / `ocv-ad-upload` / `ocv-ci-cron` / `ocv-secrets`

**Reflect:** module · crawl vs upload · live API risk · typo paths · FB/Logicad disabled?

Flag 🟠/🔴 for production Sheets/API writes, cron changes, secret rotation, unconfirmed side effects.

---

## §3 — Stage 3: `/write-spec`

Cite real module paths + workflows. Output `docs/specs/<date>-<slug>.md` (EN) if docs exist; otherwise working-tree note.

Must cover: build-time execution, `DRY_RUN`, dual notify, which ASP/API.

No Rails/Hotwire templates. DRAFT until `/check-spec`.

---

## §7 — Stage 7: `/start-coding`

**Read:** matching `ocv-*` + `batch-modules` + `sops-age-secrets` if secrets + `no-rails-assumptions`.

**Hard rules:**

- **Confirm gate:** print confirm block + wait for OK before any CONFIRM-REQUIRED action (`external-side-effect-confirm`). `DRY_RUN=1` ≠ offline (ASP still crawled). Without alert opt-in → `DISABLE_NOTIFICATIONS=1`
- No Rails / AR / controllers
- Do not rename `affliate_collection` / `schdule_build.sh`
- Module-local `common.rb` only
- Prefer `DRY_RUN=1` locally **after** confirm (mute alerts unless user allowed)
- Agent stages only; never commit/push unless asked

---

## §8 — Stage 8: `/review-staged`

1. `code-review`
2. `edge-case-boundary-review` — click-ID / empty rows / encoding
3. `ocv-ad-upload` checklist — if write paths staged
4. `ocv-ci-cron` — if workflows/Dockerfiles staged
5. `ocv-secrets` — if enc yaml / sops staged
6. `feature-cutover-orphan-review` — if disabling a network keep code
7. `pr-review-comments` — if paste-ready comments requested

**Do not** load: Hotwire, `teleme-*`, `manet-card-loan`, rails-tl-review.

**READY:** No 🔴/🟠; typo paths intact; secrets not plaintext in diff.

**Commit:** print for user; agent never commits.

---

## §9b — Stage 9b: `/create-pr`

English PR via `ocv-pr` + `workflow/pr-description-quality.mdc`.

Sections: Description · Root Cause · Solution · Impact · Risk Assessment · Testing.
