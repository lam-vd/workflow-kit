---
mode: agent
description: "Utility command to generate release summary in requester format: Summary, No 1..N requirement, and Command section. Supports language mode: /create-release (vi) or /create-release (en), default vi. Keep section keywords in English for scanability; explanation follows selected mode. Pulls content from commits/diff/spec/PR context and outputs copy-ready release notes for deployment handoff."
---

You are generating a **Release Summary** for deployment handoff.

## Language mode

This command supports optional params:

- `/create-release (vi)`
- `/create-release (en)`

If no param is provided, default to `(vi)`.

Language behavior:
- Keep section titles and structural keywords in English: `Summary`, `No 1 requirement`, `Command`.
- If mode is `(vi)`: explanations are in Vietnamese.
- If mode is `(en)`: explanations are in English.
- Domain/technical keywords remain in English in all modes.

## Task

Generate a release summary exactly in this structure:

1. `# Summary`
2. `---`
3. Repeat blocks: `## No <n> requirement`
4. Optional `## Command` section inside each requirement when relevant

Use concise, deployment-focused bullets.

## Input sources (must inspect before writing)

1. `git log <base>..HEAD --oneline`
2. `git diff --name-only <base>...HEAD`
3. Relevant spec/PR context if available (`docs/specs`, `docs/ddd`, PR description)
4. Migration/config changes from changed files

If source info is incomplete, add a short assumptions line at the end:
- `Assumptions: ...`

## Formatting rules

- Follow requirement numbering order by impact and dependency.
- Each requirement must be a clear scope chunk (BE/FE/email/UI/ops).
- Use bullets for concrete changes.
- Include rollout/dependency notes when needed (example: FE deploy after BE).
- Keep text lean; no marketing language.

## Requirement writing rules

For each `No <n> requirement` block:
- First sentence: objective of that requirement.
- Bullets: concrete shipped changes only.
- Add `## Command` only if the requirement needs runtime/deploy action.

Examples of `Command` content:
- `Need to run rails db:migrate to change the DB.`
- `Need to run bundle exec sidekiq restart.`
- `Need to clear cache after deploy.`

## Severity and risk notes

If a requirement has rollout risk, add one short bullet:
- `Risk note: ...`

If a requirement depends on another requirement's deploy order, add one short bullet:
- `Deploy order: ...`

## Hard rules

- Do NOT invent features not present in commits/diff/spec.
- Do NOT mix unrelated changes into one requirement.
- Do NOT skip migration or infra commands when required.
- Do NOT output tri-lingual blocks; output in selected mode only.
- Keep section keywords in English even in `(vi)` mode.

## Output template

```markdown
# Summary

<1-2 paragraphs summary in selected mode>

---

## No 1 requirement

<objective sentence>

- <change 1>
- <change 2>
- <change 3>

## Command

**Need to run `<command>` to <purpose>.**

---

## No 2 requirement

<objective sentence>

- <change 1>
- <change 2>

---

## No 3 requirement

<objective sentence>

- <change 1>
- <change 2>

```

## Example invocation

- `/create-release (vi)`
- `/create-release (en)`
