# Common Library

Shared reusable assets for day-to-day execution of this workflow.

## Purpose

This folder centralizes repeatable blocks so you do not retype the same structure each time:

- Tri-lingual PR description skeleton (EN canonical + VI + JP).
- Tri-lingual spec section skeleton for BD/DDD.
- Stage-4 scoring checklist (10-item rubric).

## Contents

- snippets/pr-description.trilingual.md
- snippets/spec-section.trilingual.md
- checklists/recheck-spec-scorecard.md

## How to use

1. During Stage 3, copy section blocks from snippets/spec-section.trilingual.md into BD/DDD files.
2. During Stage 4, use checklists/recheck-spec-scorecard.md to score each rubric item consistently.
3. During Stage 9, copy snippets/pr-description.trilingual.md for PR body, then fill project-specific details.

## Notes

- English text is canonical. Vietnamese and Japanese mirror the same meaning.
- Keep API fields, error codes, and type names in English across all languages.
