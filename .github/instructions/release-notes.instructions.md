---
applyTo: "docs/releases/**/*.md"
description: "Release notes formatting rules for deployment handoff documents. Enforces Summary + numbered requirement blocks + Command sections for operational steps. Keeps output concise, technical, and executable."
---

# Release Notes Rules

## Required structure

Use this order:

1. `# Summary`
2. `## No 1 requirement`
3. `## No 2 requirement`
4. ...
5. `## Command` only when there is a required operational action

## Writing style

- Keep requirement objective in one sentence.
- Use concrete bullets; avoid vague wording.
- Keep technical terms in English.
- Avoid duplicate bullets across requirement blocks.

## Scope and grouping

- Group by requirement scope (BE/FE/email/UI/ops), not by filenames.
- Do not mix unrelated changes in one requirement.
- Include deploy order note when one requirement depends on another.

## Command section policy

Include `## Command` if there is a runtime or deployment action:
- DB migration
- worker/service restart
- cache clear
- env/feature flag setup

Command sentence style:
- `Need to run <command> to <purpose>.`

## Anti-bloat policy

- Include shipped changes only.
- Do not add speculative future work.
- If a detail is uncertain, add `Assumptions:` line rather than guessing.
