---
mode: agent
description: "Stage 9b variant for ai-housemaker — Generate PR artifacts: (1) English Conventional Commits title; (2) English squash commit; (3) Japanese PR description in team format (概要 Before/After, 仕様, 対応内容, レビュワー確認項目, タスクリンク, 備考). Saves draft to ai-housemaker/docs/pr/. Refuses if /recheck-release is not READY."
---

You are at **Stage 9b: Create PR — ai-housemaker**.

> **VI**: Tạo PR cho repo ai-housemaker. Title/commit tiếng Anh; body PR tiếng Nhật theo format team (Before/After). Chỉ chạy khi `/recheck-release` READY.

## Precondition

- `/recheck-release` returned `READY ✅`. If not → STOP.
- Target repo: `ai-housemaker` (`Documents/workspaces/ai-housemaker`).
- FINAL spec exists in `docs/ddd/` or `docs/specs/`.

## Read first

1. `ai-housemaker/.cursor/rules/workflow/development-guideline.mdc` — workflow context
2. `.agents/skills/ai-housemaker-pr-description/SKILL.md` — **PR body format**
3. `ai-housemaker/.agents/skills/pr-conventions/SKILL.md` — title + squash commit only
4. `common/snippets/pr-description.ai-housemaker.ja.md` — skeleton
5. FINAL DDD: `docs/ddd/<slug>.md`
6. Reference: `docs/pr/feat-profile-ui-no92.md` if present

## Steps

1. Commit summary:
   ```sh
   git log <base>..HEAD --oneline
   git diff --stat <base>...HEAD
   ```
2. Extract from DDD: gap → After bullets; FR map → 仕様 bullets; test plan → 備考.
3. Produce **3 copy-ready blocks** + save draft.

## Output blocks

### 1. PR Title (English)

`<type>(<scope>): <imperative summary, ≤72 chars>`

### 2. Squash commit message (English)

```
<title>

<WHY paragraph + WHAT bullets>

Refs: No.XX — docs/ddd/<slug>.md
```

### 3. PR Description (Japanese)

Use exact structure from `ai-housemaker-pr-description` skill:

- `### 概要` → `Before:` / `After:` (no intro paragraph)
- Optional screenshots after After
- `仕様：` acceptance criteria
- `### 対応内容` — 4 flat checkboxes
- `### レビュワー確認項目` — 2 flat checkboxes
- `### タスクリンク`
- `### 備考`

### 4. Save draft

Write to: `ai-housemaker/docs/pr/<type>-<scope>-no<NN>.md`

## Hard rules

- DO NOT use tri-lingual collapsible PR body for ai-housemaker unless user explicitly overrides.
- DO NOT fabricate Notion/Figma links.
- Before = user pain; After = delivered outcomes (not file dump).
- Nested implementation details → 備考 only.
- Run test command if possible; put result in 備考.
- For UI PRs: remind to attach screenshots in GitHub if not in draft.

## Output format

Four sections with code fences:

1. `### Title`
2. `### Commit msg`
3. `### Description (JA)`
4. `### Draft path` — confirm saved file path
