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

## 3. Surgical Changes

- Touch only what you must.
- Match existing style.
- Remove only orphans YOUR changes created.

## 4. Goal-Driven Execution

- Define verifiable success criteria before coding.
- Loop: implement → verify → fix.
