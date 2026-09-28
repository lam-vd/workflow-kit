---
name: external-side-effect-confirm
description: >-
  Require explicit user confirmation before any command or action that can hit
  real servers, affiliate/ad platforms, Google Sheets, Slack/Google Chat/Airbrake
  alerts, or production deploys. Use before docker builds that run OCV jobs,
  ASP crawls, Ads/Yahoo uploads, Manet Airbrake-triggering runs, Thira
  deploy/reg-suit publish, workflow_dispatch, or destructive DB ops — especially
  in manet, manet-adwords-analysis--offline-cv, and thira.
---

# Confirm before external side effects

## Goal

Never surprise ops with live traffic, platform writes, or third-party alerts.
If the action can leave the laptop / local Docker network in a meaningful way
**or** notify a human channel — **stop and get an explicit OK first**.

## Hard gate

1. Classify the next action (table below).
2. If it is **CONFIRM-REQUIRED** → print the confirm block → **wait**.
3. Proceed only when the **same user message** (or a clear reply) contains an
   explicit affirm: `OK`, `confirm`, `được`, `cho chạy`, `yes`, `go ahead`
   (case-insensitive). Vague “tiếp tục” / “fix giúp” / task context alone is **not** enough.
4. If the user says `no` / `skip` / `đừng` → do not run; offer a muted / dry alternative.

**One confirm covers one planned action (or one explicitly listed batch).**
Do not reuse an older OK from a different turn for a new risky command.

## Confirm block (paste before waiting)

```text
⚠️ CONFIRM REQUIRED — external side effect

Action: <exact command or API>
Target: <ASP / Ads / Sheets / Slack / Airbrake / host / DB>
Risk: <crawl | write | notify | deploy | destructive>
Mute plan: <DISABLE_NOTIFICATIONS=1 / empty webhooks / DRY_RUN caveat / none>

Reply OK / confirm / được to proceed, or no to abort.
```

## CONFIRM-REQUIRED (non-exhaustive)

| Category | Examples |
|----------|----------|
| **OCV / ASP crawl** | `docker build` for `main/`, `affliate_collection/`, any module whose build `RUN`s crawler; login to ASP admin; scrape conversion CSVs |
| **OCV ad upload** | Google Ads / Yahoo offline conversion upload; spend sync that **writes** Sheets or Ads |
| **Notify / alert APIs** | Slack webhook, Google Chat webhook, `Airbrake.notify`, failure notify steps, health_check alerts |
| **CI that hits prod paths** | `gh workflow run`, re-run failed OCV/thira jobs, force cron locally with real secrets |
| **Manet live side effects** | Boot/run with real Airbrake project key against actions that notify; hit staging/prod URLs; mailers to real inboxes |
| **Thira publish** | `yarn deploy` intended for Netlify/Amplify; reg-suit **S3 publish**; Slack notify workflows |
| **Destructive data** | `db:drop`, `db:reset`, restore dump over shared DB, `DELETE`/`TRUNCATE` on non-scratch data, production credentials |

## Usually SAFE (no confirm) — still prefer mute when in doubt

| Action | Notes |
|--------|-------|
| Read-only git / file / code search | OK |
| `docker compose up` Manet **local** (3010/3310) with local `.env` | OK if Airbrake/Slack not firing; if unsure → confirm |
| `rspec` / `rubocop` / `yarn lint` / `yarn test` | OK |
| Edit code, stage (`git add`), print commit message | OK (no push) |
| Inspect SOPS-encrypted files **without** decrypting to run jobs | OK |

## Critical caveats (Manet-ads OCV)

- **`DRY_RUN=1` does NOT mean “no network”.** Affiliate crawlers often still **log into ASP and download data**; dry-run may only skip Ads/Sheets **upload**. Treat crawls as CONFIRM-REQUIRED.
- **`DISABLE_NOTIFICATIONS=1`** (or empty Slack/GChat webhook env) must be set **before** any local OCV build/run if the user has not explicitly allowed alerts.
- Prefer verifying mute flags in the module’s `common.rb` / env **before** asking to run.
- Typo paths are intentional (`affliate_collection`, `schdule_build.sh`) — do not “fix” them while testing.

## Manet (CMS)

- Local Foreman/Docker for UI QA is fine once DB is local.
- Avoid actions that call `Airbrake.notify` against a **real** project key unless confirmed.
- Do not point local app at staging/prod DB or CDN write paths without confirm.

## Thira (LP)

- Local `yarn start` / Storybook / Cypress **without** reg-suit publish: usually SAFE.
- `yarn deploy`, Amplify/Netlify publish, reg-suit S3 upload, Slack workflow notifies: CONFIRM-REQUIRED.

## After user confirms

1. Echo the exact command you will run.
2. Prefer muted env on the command line when the user did not opt into alerts.
3. If something still notifies unexpectedly → **stop further runs** and report.

## Anti-patterns

| Do not | Why |
|--------|-----|
| “Smoke test” OCV build because setup docs say so | Live ASP + Airbrake incident risk |
| Assume DRY_RUN = offline | Crawler still hits ASP |
| Soften confirm into a rhetorical question and run anyway | Gate failed |
| Bundle 5 risky modules under one vague OK | Scope creep |
| Push / deploy / workflow_dispatch as “part of PR help” | Needs its own confirm |

## Related

- Project rules: `.cursor/rules/security/external-side-effect-confirm.mdc` (always-on in manet / ocv / thira)
- OCV: `ocv-ci-cron`, `ocv-affiliate-crawler`, `ocv-ad-upload`, `ocv-secrets`
- Kit: `git-commit-policy` (push is separately gated)
