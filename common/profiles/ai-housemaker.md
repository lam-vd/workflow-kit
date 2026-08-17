# Project profile: ai-housemaker

> **Read order:** Load **only** the § section for the current stage. Do not read the full file every time.

> **EN**: Delta rules when working in `Documents/workspaces/ai-housemaker`
> **VI**: Quy tắc bổ sung cho repo ai-housemaker. Prompt gốc (`/analyze-task`, `/write-spec`, …) tự load file này.

## Detection

Load this profile when **any** of:

1. **Auto**: cwd or staged paths contain `ai-housemaker/`; or repo has `Gemfile` + `app/views` + Hotwire/Turbo usage
2. **Override**: user writes `(ai-housemaker)` or `/review-staged ai-housemaker` (etc.)

---

## §1 — Stage 1: `/analyze-task`

**Read first (in order):**

1. `ai-housemaker/.cursor/rules/workflow/development-guideline.mdc`
2. `.agents/skills/field-impact-analysis/SKILL.md` — if task may add/change fields
3. `ai-housemaker/AGENTS.md` — stack, tenant, Pundit

**Reflect in Stage 1 output:**

| Guideline area | Reflect in output |
|----------------|-------------------|
| Quality gates (lefthook, CI) | Estimate — lint + RSpec + pre-push overhead |
| Copilot + PM review | Estimate — review/fix cycles |
| Conventional Commits | Scope OUT — PR title/commit format unless task is CI |
| Release Please | Scope OUT — deploy/version unless task touches release |
| Tenant / Pundit / i18n JA | Impact — auth/tenant UI → often 🟠/🔴 |

**Extra hard rules:** Flag 🟠/🔴 for tenant scope leaks, Pundit/auth changes, migrations, Bedrock/external I/O, breaking API.

---

## §3 — Stage 3: `/write-spec`

**Preconditions:** Inspect actual codebase before §2 Current state — cite real file paths. UI tasks → `.agents/skills/rails-ui-layouts/SKILL.md`.

**Output files:**

| File | Language | Audience |
|------|----------|----------|
| `docs/specs/<YYYY-MM-DD>-<kebab-slug>.md` | Tri-lingual EN/VI/JP | Workflow / audit |
| `docs/ddd/<YYYY-MM-DD>-<kebab-slug>.md` | EN canonical | Implementation source of truth |
| `docs/ddd/<YYYY-MM-DD>-<kebab-slug>.vi.md` | Vietnamese only | Notion / team review |

Templates: `docs/ddd/_TEMPLATE-ai-housemaker.md`, `docs/ddd/_TEMPLATE-ai-housemaker.vi.md`

**DDD mandatory section order (EN):** Scope & phase goal → Current state → Gap → Data model → Domain/services → Routes/controllers/forms → Views & UI → HTTP errors → i18n → Security & tenant → Observability → Diagrams → Test design → Implementation order → Requirement mapping.

**Rails UI conventions:** Dashboard `layouts/dashboard`; modals Preline + Turbo Frame; auth-only `layouts/auth`; business logic in `*Manager::` services; locale default JA.

**DDD `.vi.md`:** Prose format (no.82 style); technical content must match EN; Notion-safe Mermaid (quoted messages, no `alt/else` if Notion errors).

**Hard rules:** §2 must reference real paths; every domain decision has **why** + **won't do**; DRAFT only.

---

## §4 — Stage 4: `/recheck-spec`

- Read write-spec §3 DDD mandatory sections above
- DDD `.vi.md` technical content must match EN (diagrams may be subset, not contradictory)
- BD EN = VI = JP scope alignment

---

## §7 — Stage 7: `/start-coding`

**Read first (add to base list):**

1. `ai-housemaker/.cursor/rules/workflow/development-guideline.mdc`
2. `.agents/skills/ai-housemaker-review-checklist/SKILL.md`
3. `ai-housemaker/.agents/skills/rspec-patterns/SKILL.md` — if `spec/**` changes (**HARD BAN** UI in request; lean setup)
4. `.cursor/rules/quality/erb-rubocop-lint.mdc`, `rspec-best-practices.mdc`, `implementation/views.mdc`
5. Domain rules if touched: `audited-active-storage`, `stimulus-file-input-preview`, `security/brakeman`
6. PATCH-on-change autosave → `.agents/skills/stimulus-turbo-autosave/SKILL.md`
7. List detail modal with canonical `/{index}/:id` → `.agents/skills/modal-detail-canonical-url/SKILL.md`

**Extra hard rules:**

- Every `<input>` / form field → `autocomplete` (readonly/disabled → `autocomplete="off"`)
- **Lint before `git add`:**

```bash
docker compose exec -T app bash -c 'find app/views/<feature> -name "*.html.erb" | xargs bundle exec erb_lint'
docker compose exec -T app bundle exec rubocop --force-exclusion <paths>
```

- DoD must include: **Lint clean (RuboCop + ERBLint)**

**UI layout sub-tasks:** Read `.agents/skills/rails-ui-layouts/SKILL.md` — decision tree: dashboard vs auth vs super_admin; shell → page → styles → controller → i18n → Stimulus → request specs.

**Figma SVG handoff (existing ERB):**

| Situation | Skill (kit canonical) |
|-----------|------------------------|
| Align DOM hierarchy with grouped Figma SVG export | `.agents/skills/figma-svg-html-structure/SKILL.md` |
| Audit / fix styling vs Figma SVG (only if real diff) | `.agents/skills/figma-erb-styling-audit/SKILL.md` |

Before editing markup + CSS: read `.cursor/rules/bem-css-html.mdc` (ai-housemaker symlinks under `.cursor/rules/quality/`).

**Pre-flight (forms/modals/upload):** ActiveStorage + scalar in one transaction; Stimulus 削除 clears `input.value`; request spec = status/DB/`media_type` only (**HARD BAN** body UI); nested Preline → teleport + 422 replaces modal id + backdrop cleanup + explicit `'true'`/`'false'` Stimulus bools (`hotwire-integration-patterns` PRELINE-*/TURBO-MODAL/STIMULUS-BOOL); field autosave → debounce/coalesce + option-only streams (`stimulus-turbo-autosave`, TURBO-AUTOSAVE-*); canonical detail modal URLs → lean load + filter locals on rows + no detail-frame `advance` (`modal-detail-canonical-url`, TURBO-MODAL-URL-*).

**Output add-on:**

```markdown
### Lint verification
- ERBLint: ✅ / ❌ (files: …)
- RuboCop: ✅ / ❌ (files: …)
```

---

## §8 — Stage 8: `/review-staged`

**Skills (always when profile active):**

| Skill | When |
|-------|------|
| `ai-housemaker-review-checklist` | Always — P0/P1 for staged areas (ActiveStorage, Stimulus file, request-spec scope, …) |
| `deadcode-ui-migration-review` | UI migration / deleted show→modal / orphaned partials — call-site audit before delete |
| `rails-tl-review` | Always — TL checklist (architecture / tenant / N+1 / Hotwire / RSpec); skip architecture-irrelevant nitpicks; emit **TL Summary** |
| `pr-review-comments` | When user wants **paste-ready GitHub comments** (`must`/`should`/`suggestion`/`nit` + Repro/Suggestion) — complements Full report |
| `integration-regression-review` | views / Stimulus / locales / CSS |
| `functional-verification-review` | behavior may change |
| `ai-housemaker-rspec` | `spec/**` staged → use **`ai-housemaker/.agents/skills/rspec-patterns/SKILL.md`** (**HARD BAN** UI in request; **lean setup** `build_stubbed` / small `per:`) |
| `stimulus-turbo-autosave` | Checkbox/field PATCH-on-change, debounce/coalesce, option-only streams |
| `modal-detail-canonical-url` | List detail modal gets `/{index}/:id` deep-link, filter restore, lean load |
| `hotwire-integration-patterns.mdc` | UI integration — local symlink at `ai-housemaker/.cursor/rules/quality/` |
| `paginated-list-patterns.mdc` | list/search/pagination — local symlink at `ai-housemaker/.cursor/rules/quality/` |

**Docker gates (repo root):**

```bash
docker compose exec -T app bundle exec rubocop --force-exclusion $(git diff --cached --name-only -- '*.rb')
git diff --cached --name-only -- '*.html.erb' | xargs -r docker compose exec -T app bundle exec erb_lint
docker compose exec -T app bundle exec brakeman -q --no-pager
docker compose exec -T app bundle exec rspec <related_specs>
```

Stimulus/CSS staged → `docker compose exec -T app yarn build`

**READY:** No 🔴/🟠; Brakeman exit 0; staged-related specs pass; no Hotwire / PAGE-* / TURBO-AUTOSAVE / TURBO-STREAM-HOOK P1 FAIL; **no UI asserts in request specs**.

**Commit:** Agent **never** runs `git commit` or `git push`. On READY → print copy-ready commit command; user runs in terminal (preserves author). See `git-commit-policy.mdc`. Multi-phase format: `feat(property-ui): part <N>: <area> - <summary>`.

**BLOCKED:** Brakeman warning; related spec fail; TURBO-URL/SYNC/HIST or PAGE-SEARCH/OVERFLOW FAIL; UI-in-request-spec; Manual QA FAIL.

**P0/P1 summary:** Audited ActiveStorage + scalar = one transaction; file input clear on 削除; Brakeman exit 0; **HARD BAN** UI assertions in request spec.

---

## §8 bugfix — Fix coverage (sub-task: fix / bug / regression)

When sub-task title or user intent is bugfix, run **in addition** to §8 review:

1. Identify parent: `git diff <parent>..HEAD`
2. Read `integration-regression-review` Phase E
3. Output **Fix coverage table:**

| Symptom | Root cause | Fixed by | Repro parent | Verified HEAD | Test |
|---------|------------|----------|--------------|---------------|------|

4. Verdict: **COVERED** (all symptoms) or **GAP** (🟠 finding per gap)

Every row needs concrete repro steps, not "fixed pagination".

---

## §9b — Stage 9b: `/create-pr`

**Use JA PR body** (not tri-lingual collapsible) unless user explicitly overrides.

**Read first:**

1. `ai-housemaker/.cursor/rules/workflow/development-guideline.mdc`
2. `.agents/skills/ai-housemaker-pr-description/SKILL.md`
3. `ai-housemaker/.agents/skills/pr-conventions/SKILL.md` — title + squash only
4. `common/snippets/pr-description.ai-housemaker.ja.md`
5. FINAL DDD; reference `docs/pr/feat-profile-ui-no92.md` if present

**Output:** (1) English title (2) English squash commit (3) Japanese PR body per skill (概要 Before/After, 仕様, 対応内容, レビュワー確認項目, タスクリンク, 備考) (4) Save draft to `ai-housemaker/docs/pr/<type>-<scope>-no<NN>.md`

**Hard rules:** No fabricated Notion/Figma links; Before = user pain, After = outcomes; nested details → 備考 only; **never `git push`** — generate PR text only.
