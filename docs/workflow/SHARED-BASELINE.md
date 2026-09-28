# Shared Baseline — Kit → Projects

> **EN**: What every project may symlink from `senior-workflow-kit`, vs what stays **ai-housemaker-only** or **project-local**.
> **VI**: Bộ dùng chung từ kit vs phần chỉ cho ai-housemaker / riêng từng repo.

Canonical source of truth for cross-project sync. Update this file when adding a new shared skill/rule.

---

## Layer model

```
senior-workflow-kit/          ← canonical (edit once)
  ├── SHARED (this doc)       ← symlink into Doogo / workspaces / ahm
  └── ahm-only / Hotwire      ← symlink ONLY into ai-housemaker

Doogo (Documents/workspace)   ← Doogo domain rules/skills stay local
workspaces/                   ← thin shared + ahm/suumo project skills
```

---

## ✅ SHARED — safe for Doogo + ai-housemaker (+ suumo where noted)

Stack-agnostic agent behavior and review methodology. **Symlink these.**

### Rules (`.cursor/rules/`)

| Rule | Why shared | Doogo | workspaces | ahm |
|------|------------|-------|------------|-----|
| `karpathy-guidelines.mdc` | Behavioral baseline §2.5 / §3.1 / §3.5 | ✅ symlink | ✅ copy→prefer symlink | via workspaces |
| `shared-abstraction-safety.mdc` | No feature logic in shared surfaces | ✅ symlink | ✅ symlink | via workspaces |
| `git-commit-policy.mdc` | Agent stages; user commits | ✅ symlink | optional | via kit workflow |
| `workflow/pr-review-comments.mdc` | Points to `pr-review-comments` skill for paste-ready PR comments | ✅ symlink | optional | via kit |
| `workflow/branch-peer-review.mdc` | Points to `branch-peer-review` + `/review-branch` | ✅ symlink | optional | via kit |
| `security/external-side-effect-confirm.mdc` | Confirm before live server / alert APIs | stub in kit | **alwaysApply copies** in manet / OCV / thira | optional |

### Skills (`.agents/skills/`)

| Skill | Why shared | Doogo | Notes |
|-------|------------|-------|-------|
| `karpathy-guidelines` | Thin pointer to rule | ✅ | |
| `dry-duplication-scan` | Pre-write reuse gate | ✅ | |
| `edge-case-boundary-review` | Stage-8 boundary sweep | ✅ | |
| `structural-change-analysis` | Refactor / split / parallel branches | ✅ | |
| `field-impact-analysis` | Add/change fields blast radius | ✅ | |
| `design-patterns` | Pattern cheat-sheet (use when pain exists) | ✅ | |
| `code-review` | Review report template | ✅ | Pair with Doogo `foundations/anti-patterns` for domain smells |
| `pr-review-comments` | Paste-ready GitHub PR comments (`must`/`should`/`suggestion`/`nit`) | ✅ | Complements `code-review`; use when user wants comments to post |
| `branch-peer-review` | Peer-review **another** branch: Task lock from spec+diff, then comments | ✅ | Not `/review-staged`; human prompt `docs/prompts/branch-peer-review.md` |
| `feature-cutover-orphan-review` | Partial ship / CRUD UI removed / domain kept; orphan matrix + contract sweep | ✅ | Complements UI-only `deadcode-ui-migration-review` (ahm) |
| `lean-facade-review` | Thin JSON getters, column predicates, redundant `I18n.t` `default:` | ✅ | Severity ceiling nit/suggestion; grep locales before “missing i18n” |
| `external-side-effect-confirm` | Confirm before ASP/Ads/Sheets/Slack/Airbrake/deploy/destructive DB | ✅ port_jp | Kit rule stub `alwaysApply: false`; **repo copies** with `alwaysApply: true` for manet / OCV / thira |

---

## ❌ DO NOT share into Doogo

ai-housemaker / Hotwire / Pundit / Figma-ERB stack. Wrong for Doogo (Grape + Slim/Bootstrap admin + Next/MUI FE, `PolicyHelper` not Pundit).

| Artifact | Reason |
|----------|--------|
| `ai-housemaker-*` skills | Profile-specific |
| `ai-housemaker-*.mdc` rules | Profile-specific |
| `common/profiles/ai-housemaker.md` | ahm workflow deltas |
| `figma-svg-html-structure` / `figma-erb-styling-audit` | Figma → ERB |
| `rails-ui-layouts` | Tailwind 4 / Preline layouts |
| `hotwire-integration-patterns.mdc` | Turbo/Stimulus catalog |
| `stimulus-turbo-autosave` | Hotwire PATCH-on-change |
| `modal-detail-canonical-url` | Preline list↔modal URLs |
| `deadcode-ui-migration-review` | full-page → modal cutover |
| `css-safe-text-overflow` | ahm modal/chip CSS tokens |
| `rails-tl-review` | ahm TL review rubric |
| `bem-css-html.mdc` | ahm BEM CSS |
| `paginated-list-patterns.mdc` | Hotwire/Preline list UX (not Doogo FE) |
| `pundit-policies` / housemaker-* | Wrong auth & domain |
| `integration-regression-review` | Heavily Turbo/Hotwire-oriented |
| `docs/lessons/*.md` | Post-PR incident write-ups (reference for pattern IDs) |

---

## 🟡 Optional (only if Doogo adopts kit stages)

| Artifact | When |
|----------|------|
| `writing-bd` / `writing-ddd` / `writing-ddd-integrations` | Using `/write-spec`; integrations skill when OAuth/consent/files/signed URLs/parallel provider/composite send |
| `pr-conventions` | Want tri-lingual kit PR body — **conflicts** with Doogo `doogo-pr` / `doogo-frontend-pr-description`; keep Doogo PR skills as primary |
| `create-release` | Deploy handoff in kit format |
| `functional-verification-review` | After RSpec is live on Doogo BE |
| `clean-code.mdc` / `architecture.mdc` | Only if Doogo local architecture rules are insufficient — prefer Doogo `architecture/*` |

---

## Doogo-local (never replace with kit)

| Area | Location |
|------|----------|
| Domain overview | `.cursor/rules/project/doogo-overview.mdc`, `AGENTS.md` |
| Grape / services / PolicyHelper / salon scope | `.cursor/rules/implementation/*`, `security/*` |
| FE Pages Router / MUI / Zustand | `.cursor/rules/fe/*`, `frontend-review.mdc` |
| Domain skills | `.agents/skills/backend/doogo-*`, `frontend/doogo-*` |
| Doogo PR formats | `workflow/doogo-frontend-pr-description.mdc`, `frontend/doogo-pr` |

---

## Setup (Doogo)

```bash
# From Documents/workspace
ln -sfn /path/to/senior-workflow-kit ./senior-workflow-kit

# Rules (depth: .cursor/rules → ../../ ; architecture/ → ../../../)
ln -sfn ../../senior-workflow-kit/.cursor/rules/karpathy-guidelines.mdc .cursor/rules/karpathy-guidelines.mdc
ln -sfn ../../senior-workflow-kit/.cursor/rules/shared-abstraction-safety.mdc .cursor/rules/shared-abstraction-safety.mdc
ln -sfn ../../../senior-workflow-kit/.cursor/rules/shared-abstraction-safety.mdc .cursor/rules/architecture/shared-abstraction-safety.mdc
ln -sfn ../../senior-workflow-kit/.cursor/rules/git-commit-policy.mdc .cursor/rules/git-commit-policy.mdc

# Skills (depth: .agents/skills → ../../)
for s in karpathy-guidelines dry-duplication-scan edge-case-boundary-review \
         structural-change-analysis field-impact-analysis design-patterns code-review \
         pr-review-comments branch-peer-review feature-cutover-orphan-review lean-facade-review; do
  ln -sfn ../../senior-workflow-kit/.agents/skills/$s .agents/skills/$s
done
```
---

## Drift policy

1. **Edit kit only** for SHARED rows — never fork copies in Doogo/workspaces.
2. If a project needs a path tweak, fix kit text to be path-agnostic, or add a second symlink (e.g. Doogo keeps both `.cursor/rules/shared-abstraction-safety.mdc` and `architecture/shared-abstraction-safety.mdc`).
3. New kit skill → classify here **before** symlinking into Doogo.

---

## Telemedease (`port_jp/telemedease`)

Safety-critical Rails + Grape + Slim/Vue (not Hotwire). Profile: `common/profiles/telemedease.md`.

### ✅ Symlink SHARED (same as Doogo list)

Rules: `karpathy-guidelines`, `shared-abstraction-safety`, `git-commit-policy`, optional `clean-code` / `architecture`.

Skills: `karpathy-guidelines`, `dry-duplication-scan`, `edge-case-boundary-review`, `structural-change-analysis`, `field-impact-analysis`, `design-patterns`, `code-review`, `pr-review-comments`, `branch-peer-review`, `feature-cutover-orphan-review`, `lean-facade-review`, `idor-prevention`, plus when using `/write-spec`: `writing-bd` / `writing-ddd` / **`writing-ddd-integrations`** (OAuth/consent/files/composite send).

Kit symlink from folder: `port_jp/senior-workflow-kit` → this repo.

### ❌ Do not import (ahm / Hotwire)

Same deny list as Doogo: `pundit-policies`, `stimulus-turbo-*`, `figma-*`, `rails-ui-layouts`, `bem-css`, `ai-housemaker-*`, `rails-tl-review`, etc.

### Telemedease-local (never replace with kit)

| Area | Location |
|------|----------|
| Overview / routing | `telemedease/AGENTS.md`, `.cursor/rules/project/telemedease-overview.mdc` |
| Portal isolation, PayJP, PHI | `.cursor/rules/security/*` |
| Services / Grape / CanCanCan | `.cursor/rules/implementation/*` |
| Domain skills | `.agents/skills/teleme-*` |
| EN PR body | `teleme-pr` (+ profile §9b) |
| Workflow profile | `common/profiles/telemedease.md` |

> **Note:** Some teams keep `.cursor/` / `.agents/` / `AGENTS.md` / `HERMES.md` **gitignored** (local agent config only). SHARED-BASELINE still documents the intended layout for machines that recreate Phase 0–2.

### Setup sketch

```bash
# From port_jp/
ln -sfn /path/to/senior-workflow-kit ./senior-workflow-kit
# Then under telemedease/.cursor/rules and .agents/skills — absolute or relative links to kit (see telemedease AGENTS.md)
```

---

## Skal (`port_jp/skal`)

Career media CMS (就活の未来) — Rails + Slim/Cells + Administrate + Handsaw + Pundit + Active Storage + Minitest. Profile: `common/profiles/skal.md`.

### ✅ Symlink SHARED (same list as Telemedease)

Rules: `karpathy-guidelines`, `shared-abstraction-safety`, `git-commit-policy`, optional `clean-code` / `architecture`.

Skills: `karpathy-guidelines`, `dry-duplication-scan`, `edge-case-boundary-review`, `structural-change-analysis`, `field-impact-analysis`, `design-patterns`, `code-review`, `pr-review-comments`, `branch-peer-review`, `feature-cutover-orphan-review`, `lean-facade-review`, `idor-prevention`.

### ❌ Do not import

- ai-housemaker Hotwire / Figma / Stimulus / `rails-ui-layouts`
- `teleme-*` (Grape, PayJP, PHI, CanCanCan, Refile)
- ahm `pundit-policies` as-is (different product) — use local `skal-admin` instead

### Skal-local (never replace with kit)

| Area | Location |
|------|----------|
| Overview / routing | `skal/AGENTS.md`, `.cursor/rules/project/skal-overview.mdc` |
| Admin OAuth `@theport.jp`, Pundit | `.cursor/rules/security/admin-oauth-domain.mdc`, `implementation/pundit-admin.mdc` |
| Ads CTR, Handsaw, Active Storage, services | `.cursor/rules/implementation/*` |
| Domain skills | `.agents/skills/skal-*` |
| EN PR body | `skal-pr` (+ profile §9b) |
| Workflow profile | `common/profiles/skal.md` |

> `.cursor/` / `.agents/` / `AGENTS.md` / `HERMES.md` often **gitignored** (local agent config).

---

## Manet (`port_jp/manet`)

Card-loan media CMS (ma-net.jp / Yenom) — Rails + Slim/Cells + Super admin + Pundit + Handsaw + CarrierWave + RSpec. FX domain **legacy**. Profile: `common/profiles/manet.md`.

Skills: `karpathy-guidelines`, `dry-duplication-scan`, `edge-case-boundary-review`, `structural-change-analysis`, `field-impact-analysis`, `design-patterns`, `code-review`, `pr-review-comments`, `branch-peer-review`, `feature-cutover-orphan-review`, `lean-facade-review`, `idor-prevention`, `external-side-effect-confirm`.

### ❌ Do not import

- ai-housemaker Hotwire / Figma / Stimulus / **tenant** `pundit-policies` (use local `manet-admin-cms`)
- `teleme-*` (Grape, PayJP, PHI, CanCanCan, Refile)
- skal Clearance / Active Storage / Minitest assumptions

### Manet-local

| Area | Location |
|------|----------|
| Overview / FX guard | `manet/AGENTS.md`, `.cursor/rules/project/manet-overview.mdc`, `quality/fx-legacy-guard.mdc` |
| Super auth + Pundit | `security/admin-super-isolation.mdc`, `implementation/pundit-admin.mdc` |
| Side-effect confirm (always-on) | `security/external-side-effect-confirm.mdc` + skill symlink |
| Handsaw / PC-SP / CarrierWave / services | `.cursor/rules/implementation/*` |
| Domain skills | `.agents/skills/manet-*` |
| EN PR | `manet-pr` (+ profile §9b) |
| Workflow profile | `common/profiles/manet.md` |

> Auto-detect must **exclude** `manet-adwords-analysis--offline-cv` (separate OCV batch repo).

---

## Manet-adwords OCV (`port_jp/manet-adwords-analysis--offline-cv`)

Batch Ruby monorepo — offline CV + ad spend. **Not Rails.** Jobs run at `docker build` time. Profile: `common/profiles/manet-adwords-ocv.md`.

### ✅ Symlink SHARED (same list + `external-side-effect-confirm`)

### ❌ Do not import

- `manet-*` CMS skills, `skal-*`, `teleme-*`, Hotwire, Pundit, Grape, ActiveRecord
- Cross-module shared `lib/` without DoD

### OCV-local

| Area | Location |
|------|----------|
| Overview / no-Rails | `.cursor/rules/project/ocv-overview.mdc`, `quality/no-rails-assumptions.mdc` |
| Modules / typo paths / build-time | `implementation/batch-modules.mdc` |
| SOPS | `security/sops-age-secrets.mdc` |
| Side-effect confirm (always-on) | `security/external-side-effect-confirm.mdc` + skill symlink |
| Skills | `.agents/skills/ocv-*` |
| EN PR | `ocv-pr` |
| Workflow profile | `common/profiles/manet-adwords-ocv.md` |

> Detect **before** / instead of `manet.md` when path contains `manet-adwords`.

---

## Thira (`port_jp/thira`)

Vue 2 + Webpack 4 static Manet LP (top/search/beginner) + `kw_id` A/B. **Not Rails.** Profile: `common/profiles/thira.md`.

### ✅ Symlink SHARED (same list + `external-side-effect-confirm`)

### ❌ Do not import

- `manet-*` Rails, `ocv-*`, `skal-*`, `teleme-*`, Hotwire/Figma/Next

### Thira-local

| Area | Location |
|------|----------|
| Overview | `.cursor/rules/project/thira-overview.mdc` |
| Ad params / Vue Webpack | `implementation/ad-query-params.mdc`, `vue2-static-webpack.mdc` |
| Side-effect confirm (always-on) | `security/external-side-effect-confirm.mdc` + skill symlink |
| Skills | `.agents/skills/thira-*` |
| EN PR | `thira-pr` |
| Workflow profile | `common/profiles/thira.md` |

---

## Evekatsu (`port_jp/evekatsu`)

Job-hunting event portal — Rails 8 + multi-dimension routes + PC/SP CloudFront + Pundit admin + RSpec. Profile: `common/profiles/evekatsu.md`.

### ✅ Symlink SHARED (same list)

### ❌ Do not import

- Hotwire/Figma, `teleme-*`, `thira-*`, `ocv-*`, ahm tenant pundit
- Do not rename `app/cache_clearner/`

### Evekatsu-local

| Area | Location |
|------|----------|
| Overview | `.cursor/rules/project/evekatsu-overview.mdc` |
| Routing / cache / PC-SP | `implementation/event-routing.mdc`, `event-cache.mdc`, `pc-sp-cloudfront.mdc` |
| Admin session + Pundit | `security/admin-session.mdc`, `implementation/pundit-admin.mdc` |
| Skills | `.agents/skills/eve-*` |
| EN PR | `eve-pr` |
| Workflow profile | `common/profiles/evekatsu.md` |
