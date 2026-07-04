# Common Library

Shared reusable assets for day-to-day execution of this workflow.

## Purpose

This folder centralizes repeatable blocks so you do not retype the same structure each time:

- Tri-lingual PR description skeleton (EN canonical + VI + JP).
- Tri-lingual spec section skeleton for BD/DDD.
- Stage-4 scoring checklist (10-item rubric).
- Stage-8 linter reference table.
- **Project profiles** — per-repo deltas loaded by base stage prompts (no separate slash commands).

## Contents

- `profiles/ai-housemaker.md` — ai-housemaker delta for stages 1, 3, 4, 7, 8, 9b
- `snippets/pr-description.trilingual.md`
- `snippets/pr-description.ai-housemaker.ja.md`
- `snippets/spec-section.trilingual.md`
- `checklists/recheck-spec-scorecard.md`
- `checklists/review-linters.md`

## How to use

1. During Stage 3, copy section blocks from `snippets/spec-section.trilingual.md` into BD/DDD files.
2. During Stage 4, use `checklists/recheck-spec-scorecard.md` to score each rubric item consistently.
3. During Stage 8, use `checklists/review-linters.md` for linter commands; methodology in `code-review` skill.
4. During Stage 9, copy `snippets/pr-description.trilingual.md` (or ai-housemaker JA snippet via profile §9b).
5. **ai-housemaker:** base prompts auto-load `profiles/ai-housemaker.md` § for current stage only (not full file).

6. **Context budget:** [docs/workflow/RULES-SKILLS-PROMPTS-MAP.md](../docs/workflow/RULES-SKILLS-PROMPTS-MAP.md)

## Prompt migration (deprecated commands)

| Old command | New command |
|-------------|-------------|
| `/analyze-task-ai-housemaker` | `/analyze-task` |
| `/write-spec-ai-housemaker` | `/write-spec` |
| `/start-coding-ai-housemaker` | `/start-coding` |
| `/review-staged-ai-housemaker` | `/review-staged` |
| `/review-fix-regression` | `/review-staged` (bugfix sub-task → profile §8 bugfix) |
| `/create-pr-ai-housemaker` | `/create-pr` |
| `/implement-ui-layout` | `/start-coding` + `rails-ui-layouts` skill (profile §7) |

## Notes

- English text is canonical. Vietnamese and Japanese mirror the same meaning.
- Keep API fields, error codes, and type names in English across all languages.
