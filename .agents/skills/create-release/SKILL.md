---
name: create-release
description: "Skill for writing deployment-ready release summaries in numbered requirement format: Summary + No 1..N requirement + Command sections. Optimized for engineering handoff. Enforces scope grouping, concrete change bullets, deploy-order notes, and migration/ops commands when applicable. Supports vi/en explanation mode while preserving English section keywords."
---

# Skill: Create Release Summary

## Goal

Produce a release note that is easy for engineering, QA, and ops to execute.

Format target:
- `# Summary`
- `## No 1 requirement`
- `## No 2 requirement`
- ...
- `## Command` only when required

## Data extraction order

1. Commit log (`git log <base>..HEAD --oneline`)
2. Changed files (`git diff --name-only <base>...HEAD`)
3. Migration/config/runtime files
4. PR/spec context

## Grouping rules (very important)

Group by requirement scope, not by file path.

Preferred grouping pattern:
1. BE schema/API persistence
2. FE behavior or payload sync
3. Integration handoff (FE after BE)
4. Notifications/templates
5. UI polish/fixes

Each requirement should answer one question:
- "What business/technical problem does this group solve?"

## Requirement block checklist

For each `No <n> requirement` block:
- One objective sentence.
- 2-6 concrete bullets.
- Optional risk note.
- Optional deploy-order note.
- Optional `Command` section only when action is required.

## Command section policy

Include `## Command` if and only if there is an operational step, for example:
- database migration
- background worker restart
- cache invalidation
- feature flag setup
- env var update

Command sentence style:
- `Need to run <command> to <purpose>.`

## Language mode policy

- Keep section keywords in English: `Summary`, `No n requirement`, `Command`.
- `(vi)` mode: explanations in Vietnamese.
- `(en)` mode: explanations in English.
- Keep technical terms in English in all modes.

## Anti-patterns

- Mixing unrelated fixes into one requirement block.
- Writing vague bullets like "improve system".
- Missing deploy command for migration changes.
- Reordering dependent items incorrectly (FE note before BE requirement).
- Repeating commit messages verbatim without synthesis.

## Quality bar

A good release summary must be:
- Accurate: no fabricated change.
- Executable: ops knows what to run.
- Scannable: short bullets, clean grouping.
- Dependency-aware: deploy order is explicit when needed.

## Output skeleton

```markdown
# Summary

<summary>

---

## No 1 requirement

<objective>

- <change>
- <change>

## Command

**Need to run `<command>` to <purpose>.**

---

## No 2 requirement

<objective>

- <change>
- <change>
```
