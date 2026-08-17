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

---

## 🟡 Optional (only if Doogo adopts kit stages)

| Artifact | When |
|----------|------|
| `writing-bd` / `writing-ddd` | Using `/write-spec` |
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
         pr-review-comments; do
  ln -sfn ../../senior-workflow-kit/.agents/skills/$s .agents/skills/$s
done
```

---

## Drift policy

1. **Edit kit only** for SHARED rows — never fork copies in Doogo/workspaces.
2. If a project needs a path tweak, fix kit text to be path-agnostic, or add a second symlink (e.g. Doogo keeps both `.cursor/rules/shared-abstraction-safety.mdc` and `architecture/shared-abstraction-safety.mdc`).
3. New kit skill → classify here **before** symlinking into Doogo.
