---
mode: ask
description: "Stage 1 variant for ai-housemaker — Extends /analyze-task with development-guideline.mdc: GitHub Flow, review, quality gates, naming, release constraints."
---

You are at **Stage 1: Analyze Task — ai-housemaker**.

> **VI**: Giống `/analyze-task` nhưng **bắt buộc** đọc `development-guideline.mdc` trước khi phân tích — để Scope/Impact/Estimate khớp quy trình dev team.

## Language mode

Same as `/analyze-task`: `/analyze-task-ai-housemaker (vi)` or `(en)`. Default `(vi)`.

## Read first (in order)

1. **`ai-housemaker/.cursor/rules/workflow/development-guideline.mdc`** — canonical dev workflow
2. `.agents/skills/field-impact-analysis/SKILL.md` — if task may add/change fields
3. `ai-housemaker/AGENTS.md` — stack, tenant, Pundit, domain modules

## Target repo

`Documents/workspaces/ai-housemaker` (Rails 7.2 + Hotwire + Pundit + multi-tenant).

## Output

Follow the full template and hard rules in `.github/prompts/analyze-task.prompt.md`.

Additionally reflect **development-guideline** in analysis where relevant:

| Guideline area | Reflect in Stage 1 output |
|----------------|---------------------------|
| Quality gates (lefthook, CI) | Estimate — lint + RSpec + pre-push overhead |
| Copilot + PM review | Estimate — review/fix cycles |
| Conventional Commits | Scope OUT — PR title/commit format changes unless task is about CI |
| Release Please | Scope OUT — deploy/version unless task touches release |
| Tenant / Pundit / i18n JA | Impact level — auth/tenant UI → often 🟠/🔴 |

## Hard rules (ai-housemaker)

- Same as `/analyze-task` — no code/schema/library proposals at Stage 1.
- Flag 🟠/🔴 for: tenant scope leaks, Pundit/auth changes, migrations, Bedrock/external I/O, breaking API.

## Related

- Base prompt: `.github/prompts/analyze-task.prompt.md`
- Next step: `/grooming` → `/write-spec-ai-housemaker` for spec
