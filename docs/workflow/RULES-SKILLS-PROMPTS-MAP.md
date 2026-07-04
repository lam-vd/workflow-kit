# Rules / Skills / Prompts — Linking Map

> **EN**: Single index for context budget. Each concern has **one canonical source**; other files only **pointer + when to load**.
> **VI**: Mỗi chủ đề một nguồn canonical — file khác chỉ trỏ tới, tránh đọc trùng.

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
| `karpathy-guidelines.mdc` | ~67 | Behavioral baseline (assumptions, scope, verify) |
| `git-commit-policy.mdc` | ~49 | Workflow gate — when commit is allowed |

**Not** always-on (load via globs or stage prompt): `clean-code.mdc`, `architecture.mdc`.

---

## By workflow stage

| Stage | Prompt | Read (in order) | Do not re-read |
|-------|--------|-----------------|----------------|
| 1 | `analyze-task` | `field-impact-analysis` skill if fields; profile §1 if ai-housemaker | — |
| 2 | `grooming` | — | — |
| 3 | `write-spec` | `writing-bd`, `writing-ddd`; profile §3; `rails-ui-layouts` if UI | — |
| 4 | `recheck-spec` | `common/checklists/recheck-spec-scorecard.md`; profile §4 | Full DDD in prompt |
| 5 | `check-spec` | — | — |
| 6 | `breakdown-task` | — | — |
| 7 | `start-coding` | profile §7; `design-patterns` if needed; rules apply via **globs** on edited files | Duplicate coding table in prompt |
| 8 | `review-staged` | **`code-review` skill** (report template); `review-linters`; supplements **only if staged paths match** | 8-dimension prose in prompt |
| 8 + UI | (above) | `integration-regression-review` + `hotwire-integration-patterns` | Re-list every TURBO-* in prompt |
| 8 + list | (above) | `paginated-list-patterns` | — |
| 8 + behavior | (above) | `functional-verification-review` | — |
| 8 + ai-housemaker | (above) | profile §8; `ai-housemaker-review-checklist` | `ai-housemaker-review-patterns` body (pointer only) |
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
| Linters table | `common/checklists/review-linters.md` | — |
| Hotwire pattern catalog | `hotwire-integration-patterns.mdc` | `integration-regression-review` skill |
| Pagination patterns | `paginated-list-patterns.mdc` | profile §8, checklist skill |
| ai-housemaker P0/P1 | `ai-housemaker-review-checklist` skill | `ai-housemaker-review-patterns.mdc` |
| ai-housemaker delta | `common/profiles/ai-housemaker.md` | per-stage prompt profile block |
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
2. Load **integration / functional / pagination** skills only when staged file types match.
3. Do **not** read both `ai-housemaker-review-patterns.mdc` and checklist skill — **checklist skill only**.
4. At Stage 9: ai-housemaker → **stop** after profile §9b + JA skill; do not load tri-lingual snippet.
