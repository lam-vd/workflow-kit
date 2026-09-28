# Rules / Skills / Prompts — Linking Map

> **EN**: Single index for context budget. Each concern has **one canonical source**; other files only **pointer + when to load**.
> **VI**: Mỗi chủ đề một nguồn canonical — file khác chỉ trỏ tới, tránh đọc trùng.

**Cross-project sync (Doogo ↔ kit ↔ ai-housemaker):** see [SHARED-BASELINE.md](./SHARED-BASELINE.md) — what to symlink into Doogo vs ahm-only Hotwire/Pundit/Figma artifacts.

## Layer model

```
Prompt (.github/prompts/)     → steps + gates + "read X"
  ↓
Skill (.agents/skills/)       → methodology + output templates
  ↓
Rule (.cursor/rules/)         → always-on baseline OR glob-triggered catalogs
  ↓
common/                       → profiles, checklists, snippets (no logic duplication)
```

**Do not** embed full templates in prompts if the same template lives in a skill or `common/snippets/`.

---

## Always apply (small baseline only)

| Rule | Lines | Why always |
|------|-------|------------|
| `karpathy-guidelines.mdc` | ~90 | Behavioral baseline (assumptions, scope, **§3.1 preserve existing logic**, verify) |
| `git-commit-policy.mdc` | ~49 | Workflow gate — when commit is allowed |
| `shared-abstraction-safety.mdc` | ~60 | Stop feature-only logic in shared helpers/controllers (cross-screen regressions) |

**Not** always-on (load via globs or stage prompt): `clean-code.mdc`, `architecture.mdc`.

---

## By workflow stage

| Stage | Prompt | Read (in order) | Do not re-read |
|-------|--------|-----------------|----------------|
| 1 | `analyze-task` | **`dry-duplication-scan`** if task adds behavior; `field-impact-analysis` if fields; `structural-change-analysis` if refactor/split/parallel branches; profile §1 if ai-housemaker | — |
| 2 | `grooming` | `structural-change-analysis` if refactor scope still open | — |
| 3 | `write-spec` | `writing-bd`, `writing-ddd`, `writing-ddd-integrations` (OAuth/consent/files/composite send); profile §3; `rails-ui-layouts` if UI | — |
| 4 | `recheck-spec` | `common/checklists/recheck-spec-scorecard.md`; profile §4 | Full DDD in prompt |
| 5 | `check-spec` | — | — |
| 6 | `breakdown-task` | — | — |
| 7 | `start-coding` | **`dry-duplication-scan`** (Reuse Scan row per new symbol); profile §7; `design-patterns` if needed; Figma SVG → `figma-svg-html-structure` / `figma-erb-styling-audit`; detail overflow → `css-safe-text-overflow`; rules apply via **globs** on edited files | Duplicate coding table in prompt |
| 8 | `review-staged` | **`code-review` skill** (report template); **`edge-case-boundary-review`** (§1d, unless copy/CSS-only); `review-linters`; supplements **only if staged paths match** | 8-dimension prose in prompt |
| 8 peer | `review-branch` | **`branch-peer-review`** + `pr-review-comments`; Task lock from spec+diff; **no** commit command | `/review-staged` Full report / commit gate |
| 8 + new symbols | (above) | `dry-duplication-scan` **reverse pass** (`DUP-*`) | Re-running the Stage 1 scan verbatim |
| 8 + UI | (above) | `integration-regression-review` + `hotwire-integration-patterns`; overflow → `css-safe-text-overflow`; UI migration orphans → `deadcode-ui-migration-review`; autosave → `stimulus-turbo-autosave`; canonical detail modal URLs → `modal-detail-canonical-url` | Re-list every TURBO-* in prompt |
| 8 + cutover / partial ship | (above) | `feature-cutover-orphan-review` (orphan matrix + `CONTRACT-*` sweep) | CRUD UI gone / domain kept; new job·channel·FE template |
| 8 + thin model/API facade | (above) | `lean-facade-review` (JSON getters, predicates, `I18n.t` `default:`) | Keep view APIs; grep locales before “missing i18n” |
| 8 + list | (above) | `paginated-list-patterns` | — |
| 8 + behavior | (above) | `functional-verification-review` | — |
| 8 + ai-housemaker | (above) | profile §8; `ai-housemaker-review-checklist`; **`rails-tl-review`** (TL Summary); `ai-housemaker-rspec` **HARD BAN** UI-in-request; UI migration deadcode → `deadcode-ui-migration-review` | `ai-housemaker-review-patterns` body (pointer only) |
| 9a | `recheck-release` | — | — |
| 9b | `create-pr` | profile §9b **or** `pr-conventions` skill + `snippets/pr-description.trilingual.md` | Tri-lingual template inline in prompt |
| 9c | `create-release` | `create-release` skill | — |

---

## Canonical source per concern

| Concern | Canonical | Thin pointers |
|---------|-----------|---------------|
| Commit when allowed | `git-commit-policy.mdc` | `pr-conventions` skill, `code-review` skill |
| PR title / tri-lingual body | `pr-conventions` skill + snippet | `pr-conventions.mdc`, `create-pr` prompt |
| ai-housemaker PR (JA) | `ai-housemaker-pr-description` skill + snippet | `ai-housemaker-pr-description.mdc`, profile §9b |
| Code style | `clean-code.mdc` (globs) | `code-review` dimension #2 |
| Layering | `architecture.mdc` (globs) | `code-review` dimension #3 |
| Review report format | `code-review` skill § Full report template | `review-staged` prompt |
| Peer review other branch | `branch-peer-review` skill | `review-branch` prompt; `docs/prompts/branch-peer-review.md`; `pr-review-comments` for blocks |
| Linters table | `common/checklists/review-linters.md` | — |
| Hotwire pattern catalog | `hotwire-integration-patterns.mdc` | `integration-regression-review` skill |
| Stimulus field/checkbox autosave | `stimulus-turbo-autosave` skill | TURBO-AUTOSAVE-* / TURBO-STREAM-HOOK-01 in hotwire rule |
| Canonical detail modal URL (list↔modal) | `modal-detail-canonical-url` skill | TURBO-MODAL-URL-01..06 in hotwire rule; ai-housemaker **symlink** `.agents/skills/modal-detail-canonical-url` |
| ai-housemaker RSpec layers | `ai-housemaker/.agents/skills/rspec-patterns/SKILL.md` (**HARD BAN** UI in request) | kit `ai-housemaker-rspec`; `rspec-best-practices.mdc` |
| Safe text / chip overflow | `css-safe-text-overflow` skill (kit) | CSS-OVERFLOW-01 in `hotwire-integration-patterns`; INT-CSS-02; ai-housemaker **symlink** `.agents/skills/css-safe-text-overflow` |
| UI migration deadcode (show→modal) | `deadcode-ui-migration-review` skill (kit) | Stage 8 when deleting views / modal-only cutover; ai-housemaker **symlink** `.agents/skills/deadcode-ui-migration-review` |
| Feature cutover / partial ship orphans | `feature-cutover-orphan-review` skill (kit, **SHARED**) | CRUD UI removed / domain kept; enqueue·job·channel; FE template never read; `CONTRACT-*` |
| Lean facade / thin getters / I18n `default:` | `lean-facade-review` skill (kit, **SHARED**) | JSON passthrough; `col == CONST` predicates; grep locales before “missing i18n” |
| Pagination patterns | `paginated-list-patterns.mdc` | profile §8, checklist skill |
| DRY / duplicate logic per layer | `dry-duplication-scan` skill (`DUP-*`) | karpathy §2.5; `analyze-task` step 4; `start-coding` step 3; `code-review` Lean gates |
| Edge cases / boundary values | `edge-case-boundary-review` skill (`EDGE-*`) | `review-staged` step 4a; `code-review` dimension #4 + §1d; `writing-ddd` Edge cases |
| Shared helper/controller blast radius | `shared-abstraction-safety.mdc` | karpathy §3.5; PAGE-SHARED-01 in paginated-list |
| ai-housemaker P0/P1 | `ai-housemaker-review-checklist` skill | `ai-housemaker-review-patterns.mdc` |
| ai-housemaker TL criteria | `rails-tl-review` skill (Critical / Suggestions / Praise) | `review-staged` step 4b; profile §8 |
| ai-housemaker delta | `common/profiles/ai-housemaker.md` | per-stage prompt profile block |
| telemedease delta | `common/profiles/telemedease.md` | portals / PayJP / Grape / EN PR; prompts load §1/§3/§7/§8/§9b |
| skal delta | `common/profiles/skal.md` | ads CTR / Handsaw / Pundit / Active Storage / Minitest / EN PR; prompts load §1/§3/§7/§8/§9b |
| manet delta | `common/profiles/manet.md` | card_loan / Super / Handsaw / Pundit / CarrierWave / RSpec / FX legacy; EN PR; detect excludes manet-adwords |
| manet-adwords-ocv delta | `common/profiles/manet-adwords-ocv.md` | batch OCV / build-time Docker / Sheets / ASP / SOPS; `ocv-*` skills; not Rails |
| thira delta | `common/profiles/thira.md` | Vue2 LP / kw_id A/B / Cypress-reg-suit; not Rails manet |
| evekatsu delta | `common/profiles/evekatsu.md` | event routes / PC-SP CloudFront / cache / Pundit / RSpec; `eve-*` skills |
| Figma SVG → ERB structure | `figma-svg-html-structure` skill (kit) | profile §7; ai-housemaker **symlink** `.agents/skills/figma-svg-html-structure` |
| Figma SVG → ERB styling audit | `figma-erb-styling-audit` skill (kit) | profile §7; ai-housemaker **symlink** `.agents/skills/figma-erb-styling-audit` |
| BEM HTML + feature CSS | `.cursor/rules/bem-css-html.mdc` (kit) | ai-housemaker **symlink** `quality/bem-css-html.mdc`; Figma SVG skills |
| Karpathy behavior | `karpathy-guidelines.mdc` | `karpathy-guidelines` skill (summary) |

---

## Glob-triggered rules (context only when relevant)

| Rule | Triggers when editing |
|------|------------------------|
| `clean-code.mdc` | `*.{rb,ts,tsx,js,py,go,java,kt,rs,php,cs}` |
| `architecture.mdc` | same code globs |
| `hotwire-integration-patterns.mdc` | views, Stimulus, locales, feature CSS |
| `paginated-list-patterns.mdc` | controllers, list partials, pagination |
| `pr-conventions.mdc` | optional; skill preferred at Stage 9 |
| `ai-housemaker-pr-description.mdc` | `**/ai-housemaker/docs/pr/**` |
| `ai-housemaker-review-patterns.mdc` | pointer only — use checklist skill |

---

## Multi-root note (workspace + workspaces + kit)

If `karpathy-guidelines` exists in multiple roots, content is identical — harmless duplicate context. Prefer **one** kit copy when working in kit-only mode.

---

## Agent discipline

1. At Stage 8: read **`code-review` skill once** — emit report from its template.
2. Peer / other-dev branch → **`branch-peer-review`** + `/review-branch` — do not emit Stage-8 commit command.
3. Load **integration / functional / pagination** skills only when staged file types match.
4. Do **not** read both `ai-housemaker-review-patterns.mdc` and checklist skill — **checklist skill only**.
5. At Stage 9: ai-housemaker → **stop** after profile §9b + JA skill; do not load tri-lingual snippet.
