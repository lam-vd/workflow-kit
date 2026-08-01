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

## 3.5. Shared Surface Blast Radius

- Do not fix one screen by changing shared helpers/controllers used by many.
- Prefer call-site or opt-in; keep defaults for other consumers.
- Detail: `.cursor/rules/shared-abstraction-safety.mdc`

## 4. Goal-Driven Execution

- Define verifiable success criteria before coding.
- Loop: implement → verify → fix.
