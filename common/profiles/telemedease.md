# Project profile: telemedease

> **Read order:** Load **only** the § section for the current stage.
>
> **EN**: Delta rules for `Documents/workspace/port_jp/telemedease` (and Hermes cwd under that tree).
> **VI**: Quy tắc bổ sung cho Telemedease — 4 portal, Grape, CanCanCan, PayJP. Không dùng profile ai-housemaker (Hotwire/Pundit).

## Detection

Load this profile when **any** of:

1. **Auto**: cwd or staged paths contain `telemedease/` or `port_jp/telemedease`
2. **Override**: user writes `(telemedease)` or `/review-staged telemedease` (etc.)

**Do not** load `common/profiles/ai-housemaker.md` for this repo.

---

## §1 — Stage 1: `/analyze-task`

**Read first (in order):**

1. `telemedease/AGENTS.md` — portals, PayJP, Diagnosis lifecycle, skills routing
2. `telemedease/.cursor/rules/project/telemedease-overview.mdc`
3. `.agents/skills/dry-duplication-scan/SKILL.md` — if task adds behavior
4. `.agents/skills/field-impact-analysis/SKILL.md` — if fields may change
5. Domain skill if obvious: `teleme-portals` / `teleme-grape-api` / `teleme-payment` / `teleme-diagnosis-flow`

**Reflect in Stage 1 output:**

| Area | Reflect |
|------|---------|
| Portal (`pub`/`pat`/`dct`/`mng`) | Which portal(s); session isolation risk |
| API | v1 / v2 / fhir — mobile contract |
| PayJP / PHI | Often 🟠/🔴 |
| CanCanCan / Ability | Authz blast radius |
| HMS / Omron | EOL — confirm before extending |

**Extra hard rules:** Flag 🟠/🔴 for payment, Ability, portal session, migrations touching PHI, unscoped diagnosis/patient queries.

---

## §3 — Stage 3: `/write-spec`

**Preconditions:** Cite real paths under `telemedease/` (controllers, apis, services).

**Read first (add to base `writing-bd` / `writing-ddd`):**

1. `.agents/skills/writing-ddd-integrations/SKILL.md` — when the task touches **any** of: OAuth/OIDC provider link, consent/opt-in before PII, chat/clinical file upload/download, signed URLs, parallel provider copy (leave live Aizu/PHR untouched), composite “save + partner send”
2. `teleme-refile-attachments` — when Refile / chat mime / custom backend / replace-one-slot files (implementation IDs; still cite in DDD FILE/REFILE notes)

**Output (default):**

| File | Language |
|------|----------|
| `docs/specs/<YYYY-MM-DD>-<kebab-slug>.md` | EN canonical (+ VI mirror OK if team wants) |

No ai-housemaker `.vi.md` / Preline / Hotwire templates.

**Must cover when relevant:** portal(s), Ability/API auth, PayJP hospital scoping, Diagnosis `process_status` (`interview` → `treatment` → `terminated` only), API version; for integrations — CUR retention vs link expiry, FILE auth lookup for new STI/types, CON consent-before-retrieve, CMP fail-closed, SEC access vs residual (not “no risk”).

**Hard rules:** DRAFT until `/check-spec`; no invented 5-stage diagnosis funnel; do not invent partner API hosts/secrets — CONTRACT-LEVEL + TBD OK.

---

## §7 — Stage 7: `/start-coding`

**Read first (add to base list):**

1. `telemedease/.cursor/rules/security/portal-isolation.mdc` — if web portal
2. Matching `teleme-*` skill (`teleme-services`, `teleme-grape-api`, `teleme-payment`, …)
3. `telemedease/.cursor/rules/security/payjp-safety.mdc` — if payment
4. `telemedease/.cursor/rules/implementation/cancancan.mdc` — if Ability/authz
5. `teleme-refile-attachments` + `implementation/refile-attachments.mdc` — if Refile / `attachment :file` / custom backend / chat file mime
6. `dry-duplication-scan` — Reuse Scan row before each new symbol

**Extra hard rules:**

- No Pundit / Stimulus / Turbo patterns
- Match existing service idioms (`ActiveModel::Model#execute` or PORO) — no new `ApplicationService` unless DoD
- Refile: no storage-deleting `destroy*` inside a txn that can still fail (REFILE-TX-01); honor stub chat DoD (CONTRACT-STUB-01)
- Lint/test in Docker when compose is used:

```bash
docker compose exec -T rails bundle exec rubocop --force-exclusion <paths>
docker compose exec -T rails bundle exec rspec <paths>
```

- Agent **stages only** (`git add`); never `git commit` / `git push` unless user explicitly asks

---

## §8 — Stage 8: `/review-staged`

**Skills (when profile active):**

1. `code-review` (report template)
2. `edge-case-boundary-review` — unless copy/CSS-only
3. `idor-prevention` — if controllers/apis/Ability touch IDs
4. `teleme-payment` checklist — if payment staged
5. `teleme-refile-attachments` — if Refile / attachments / chat file mime staged (REFILE-TX / CONTRACT-STUB / MIME / nosniff)
6. `feature-cutover-orphan-review` — if removing HMS/UI keep domain **or** stub→full delivery cutover
7. `lean-facade-review` — if thin getters / I18n `default:`
8. `pr-review-comments` — if user wants paste-ready GitHub comments

**Do not** load: `rails-tl-review`, `ai-housemaker-review-checklist`, Hotwire integration skills, HARD BAN UI-in-request (ahm-only).

**READY:** No 🔴/🟠; related specs pass; portal/PayJP/PHI checks OK.

**Commit:** On READY → print copy-ready `git commit` for **user**; agent never commits.

---

## §9b — Stage 9b: `/create-pr`

**English PR body** (not JA ai-housemaker format).

**Read:** `telemedease/.agents/skills/teleme-pr/SKILL.md` + `workflow/pr-description-quality.mdc`

Required sections: Description · Root Cause · Solution · Impact · Risk Assessment · Testing.

Title + commits: English Conventional Commits.
