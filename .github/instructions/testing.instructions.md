---
applyTo: "**/*.{test,spec}.{ts,tsx,js,jsx,py,go,java,kt,rb}"
description: "Testing rules applied to every test file. Enforces test pyramid (many unit, fewer integration, minimal e2e), behavior-focused test names ('should <behavior> when <condition>'), strict AAA structure (Arrange-Act-Assert), coverage targets per layer (Domain ≥90%, Application ≥80%, Infrastructure smoke+contract, UI critical-flow e2e), boundary-only mocking (mock HTTP/DB/time/random — never domain), and explicit edge-case coverage (empty/null, boundary values, concurrency, dependency error paths, i18n/unicode/timezone). Flags 🟠 anti-patterns: order-dependent tests, real network/DB without cleanup, conditional logic inside tests, placeholder asserts, meaningless test names."
---

# Testing Rules

## Pyramid
- **Unit** (many): pure logic, branches, edge cases.
- **Integration** (medium): module + DB + external mocks.
- **E2E** (few): happy path + 1–2 critical user flows.

## Naming
- Test names describe behavior, NOT implementation.
- Format: `should <expected behavior> when <condition>`.
- Example: `should return 400 when email is empty`.

## Structure (AAA)
```
// Arrange
const input = ...
// Act
const result = sut(input)
// Assert
expect(result).toBe(...)
```

## Coverage targets
- Domain layer: ≥ 90%.
- Application: ≥ 80%.
- Infrastructure: smoke + contract tests.
- UI: critical flows via e2e.

## Mocking
- Mock at boundaries (HTTP, DB, time, randomness) — DO NOT mock domain.
- Avoid complex mock chains; if needed → refactor production code.

## Edge case checklist (per business-logic function)
- [ ] Empty / null / undefined input
- [ ] Boundary values (0, max int, empty array, single element)
- [ ] Concurrent / race conditions (if async)
- [ ] Error paths from dependencies
- [ ] i18n / unicode / timezone

## Anti-patterns (flag 🟠)
- Tests dependent on execution order.
- Tests using real network / real DB without cleanup.
- Tests with `if/else` inside test logic (split into separate tests).
- `expect(true).toBe(true)` or placeholder asserts.
- Meaningless test names (`test1`, `should work`).
