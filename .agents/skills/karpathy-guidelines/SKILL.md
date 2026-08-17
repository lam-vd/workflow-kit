---
name: karpathy-guidelines
description: Behavioral guidelines to reduce common LLM coding mistakes. Use when writing, reviewing, or refactoring code to avoid overcomplication, make surgical changes, surface assumptions, and define verifiable success criteria.
---

# Karpathy Guidelines

Behavioral guidelines to reduce common LLM coding mistakes.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

Full rule (auto-applied in Cursor): `.cursor/rules/karpathy-guidelines.mdc`

## 1. Think Before Coding

- State assumptions explicitly. If uncertain, ask.
- Present multiple interpretations — don't pick silently.
- Push back when a simpler approach exists.

## 2. Simplicity First

- Minimum code that solves the problem. Nothing speculative.
- No abstractions for single-use code.

## 2.5. Search Before Implement

- Grep before creating any new symbol — reuse beats a parallel copy.
- No new symbol without a recorded search; ≥3 layers with zero hits ⇒ genuinely new.
- Detail: `.agents/skills/dry-duplication-scan/SKILL.md`

## 3. Surgical Changes

- Touch only what you must.
- Match existing style.
- Remove only orphans YOUR changes created.

## 3.1. Preserve existing logic (task-scoped only)

- DoD only — do not rewrite working legacy paths "while here".
- Prefer stop-calling / call-site guard over hollowing a service into a no-op.
- After removing call sites: **grep leftovers**; if zero hits → warn user (dead code: keep / delete now / follow-up).
- Keep control-flow idioms (redirects, signatures, find order) unless DoD forces change.
- No prophylactic hardening — if a special case seems needed, **ask the user first**.
- Every staged hunk must map to a DoD bullet; revert taste-only diffs.
- On Ruby files: **preserve** existing `# frozen_string_literal: true` if present. Add on **new** files. Do not force-add onto legacy files without verifying no string-literal mutation (or user/DoD asks).

## 3.5. Shared Surface Blast Radius

- Do not fix one screen by changing shared helpers/controllers used by many.
- Prefer call-site or opt-in; keep defaults for other consumers.
- Detail: `.cursor/rules/shared-abstraction-safety.mdc`

## 4. Goal-Driven Execution

- Define verifiable success criteria before coding.
- Loop: implement → verify → fix.
