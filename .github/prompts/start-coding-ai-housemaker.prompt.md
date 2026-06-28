---
mode: agent
description: "Stage 7 variant for ai-housemaker — Extends /start-coding with RuboCop + ERBLint gates, autocomplete on inputs, and scoped lint commands before staging."
---

You are at **Stage 7: Start Coding — ai-housemaker**.

> **VI**: Giống `/start-coding` nhưng **bắt buộc** chạy RuboCop + ERBLint trên file đã sửa trước khi `git add`. Tránh fail CI job `static-analysis`.

## Preconditions

Same as `/start-coding` — spec `FINAL`, sub-task list exists.

Target repo: `Documents/workspaces/ai-housemaker`.

## Read first (in order)

1. `docs/ddd/<task>.md` + sub-task DoD
2. `ai-housemaker/.cursor/rules/workflow/development-guideline.mdc` — quality gates & workflow
3. `.cursor/rules/clean-code.mdc` (senior-workflow-kit) OR project conventions
4. `.cursor/rules/architecture.mdc`
5. **`.agents/skills/static-analysis-lint/SKILL.md`** — RuboCop + ERBLint checklist
6. **`.cursor/rules/quality/erb-rubocop-lint.mdc`** — autocomplete quick reference
7. **`.cursor/rules/implementation/views.mdc`** — nếu sub-task có ERB
8. Domain skill nếu cần: `rails-controllers`, `housemaker-services`, `rspec-patterns`

## Extra hard rules (ai-housemaker)

- **Mọi `<input>` / `text_field` / `email_field` / `password_field` / `telephone_field` phải có `autocomplete`.**
  - Readonly/disabled display → `autocomplete="off"`.
- Sau khi code xong, **chạy lint trước stage**:

```bash
# ERB đã sửa (thay path)
docker compose exec -T app bash -c 'find app/views/<feature> -name "*.html.erb" | xargs bundle exec erb_lint'

# Ruby đã sửa
docker compose exec -T app bundle exec rubocop --force-exclusion <paths>
```

- Nếu lint fail → fix trước khi `git add`. Không commit trong stage này.
- DoD sub-task phải có dòng: **Lint clean (RuboCop + ERBLint)**.

## Output (thêm vào template /start-coding)

```markdown
### Lint verification
- ERBLint: ✅ / ❌ (files: …)
- RuboCop: ✅ / ❌ (files: …)
```

## Related

- Base workflow: `senior-workflow-kit/.github/prompts/start-coding.prompt.md`
- CI gates: `.cursor/rules/workflow/quality-gates.mdc`
- After sub-task: `/review-staged`
