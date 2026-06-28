---
name: ai-housemaker-pr-description
description: "Skill for writing ai-housemaker Pull Request descriptions in the team's Japanese PR body format (概要 Before/After, 仕様, 対応内容, レビュワー確認項目, タスクリンク, 備考). Use when creating or updating PR descriptions for the ai-housemaker repo, saving drafts to docs/pr/, or when the user asks for No.XX-style PR format. PR title and squash commit remain English Conventional Commits per pr-conventions skill."
---

# Skill: ai-housemaker PR Description

> **Canonical workflow:** `ai-housemaker/.cursor/rules/workflow/development-guideline.mdc` — PR title EN (Conventional Commits), body JA (this skill).

## When to use

- Creating PR description for **ai-housemaker** (`/Users/lamship/Documents/workspaces/ai-housemaker`)
- User references `docs/pr/*.md`, No.XX task format, or meeting-history / profile PR examples
- Stage 9b (`/create-pr`) when target repo is ai-housemaker and team uses **Japanese PR body** (not tri-lingual collapsible)

## Language split

| Artifact | Language |
|----------|----------|
| PR title | English — Conventional Commits (`feat(profile): ...`) |
| Squash commit message | English — see `pr-conventions` skill |
| PR description body | **Japanese** — template below |
| Chat with team | Vietnamese (optional summary for author) |

## Output locations

| Purpose | Path |
|---------|------|
| Draft before `gh pr create` | `ai-housemaker/docs/pr/<type>-<scope>-no<NN>.md` |
| GitHub PR body | Copy from draft; attach screenshots in GitHub UI |

Filename pattern: `feat-profile-ui-no92.md`, `fix-meeting-delete-no85.md`

## Required structure (exact section order)

```markdown
### 概要

Before:

- <pain point 1 — user-facing, past tense>
- <pain point 2>
- <pain point 3–4>

After:

- <deliverable 1 — outcome, not file list>
- <deliverable 2>
- <deliverable 3–5>

<!-- Optional: screenshots between After and 仕様 -->
<img width="..." height="..." alt="..." src="https://github.com/user-attachments/assets/..." />

仕様：

- <functional requirement / acceptance criterion>
- <behavior rule from DDD or ticket>
- <permission / tenant / edge-case rule if relevant>

### 対応内容
- [ ] 実装
- [ ] 単体テスト
- [ ] featureテスト
- [ ] 動作確認

### レビュワー確認項目
- [ ] 動作確認
- [ ] コードレビュー

### タスクリンク
- [No.XX — <title>](<notion-url>)
- Spec: `docs/ddd/<file>.md`
- Figma: <url> （node ids if UI）

### 備考
- <test evidence, file stats, out-of-scope files>
- <known follow-ups 🟡>
- <rollback plan if 🟠/🔴>
```

## Section rules

### 概要 — Before / After

- **No opening summary paragraph** before `Before:` — go straight to bullets (team convention).
- **Before**: 3–4 bullets describing **problems / gaps** from user or reviewer perspective. Past state.
- **After**: 3–5 bullets describing **outcomes** delivered. Present perfect / できるようにしました.
- Write for PM/designer readability; avoid dumping class names unless necessary for disambiguation.
- **Do not** mix implementation file lists into After — move to 備考.

### Screenshots

- Place **after** `After:` block, **before** `仕様：`
- Use GitHub user-attachments URLs when available
- Cover: desktop + mobile for UI PRs; key modals / slide-overs / error states
- Omit section if no images yet; note in 備考: `スクリーンショットは PR 作成時に添付`

### 仕様：

- Bullet list of **acceptance criteria** — what the system must do
- Pull from FINAL DDD FR map / ticket; not a commit log
- Include: routes, permissions (Pundit), tenant scope, data formatting rules, out-of-scope explicitly

### 対応内容

- **Four top-level checkboxes only** — no nested bullets under each item
- Mark `[x]` only when actually done; leave `[ ]` for pending QA (e.g. designer sign-off)
- Details (spec paths, example counts) go in **備考**

### レビュワー確認項目

- **Two top-level checkboxes**: 動作確認 / コードレビュー
- Optional: add nested checklist under 備考 or separate reviewer guide — not in this section per team template

### タスクリンク

- Notion task link with `No.XX` in label
- Relative spec path under `docs/ddd/` or `docs/specs/`
- Figma URL + node ids for UI tasks

### 備考

- Test commands and results (`bundle exec rspec`, example counts)
- Main files / diff stats (table or bullets)
- Files intentionally excluded from PR
- Known follow-ups with 🟡
- Rollback steps for 🟠/🔴 impact

## Workflow integration

1. Read FINAL spec: `docs/ddd/<slug>.md`
2. Run `git log <base>..HEAD --oneline` and `git diff --stat <base>...HEAD`
3. Read `.agents/skills/pr-conventions/SKILL.md` for **title + commit message only**
4. Apply **this skill** for PR body
5. Save draft to `docs/pr/<slug>.md`
6. User runs `gh pr create` with English title + Japanese body

## ai-housemaker specifics

| Topic | Convention |
|-------|------------|
| Auth | Devise `TenantUser` |
| Authorization | Pundit — mention policy in 仕様 when behavior is role-gated |
| Tenant | `Current.tenant` / `for_current_tenant` — call out scope risks in 備考 |
| UI | ERB + Hotwire + Tailwind 4 + Preline; Figma link required for UI PRs |
| Services | `ApplicationService` + `.call`; list new services in 備考 not After |
| Tests | RSpec — `docker compose exec -e RAILS_ENV=test app bundle exec rspec` |
| i18n | User-facing strings JA via `config/locales/` |

## Anti-patterns

- Opening paragraph before `Before:` when team template forbids it
- Nested bullets under 対応内容 top-level items
- English PR body when team expects Japanese (unless user explicitly asks EN)
- Fabricated Notion/Figma links
- Listing every changed file in After bullets
- Marking 動作確認 `[x]` when designer QA is still pending

## Reference examples

- `ai-housemaker/docs/pr/feat-profile-ui-no92.md` — profile UI (No.92)
- Meeting history PR (No.82) — slide-over + modals + soft delete pattern

## Related files

- `ai-housemaker/.cursor/rules/workflow/development-guideline.mdc` — canonical dev workflow
- `.github/prompts/create-pr-ai-housemaker.prompt.md` — agent prompt for Stage 9b variant
- `.cursor/rules/ai-housemaker-pr-description.mdc` — rule summary
- `common/snippets/pr-description.ai-housemaker.ja.md` — copy-paste skeleton
- `.agents/skills/pr-conventions/SKILL.md` — title + squash commit (English)
