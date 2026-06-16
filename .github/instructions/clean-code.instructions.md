---
applyTo: "**/*.{ts,tsx,js,jsx,py,go,java,kt,rb,rs,php,cs}"
description: "Clean code rules auto-applied to every source file across mainstream languages (TS/JS, Python, Go, Java/Kotlin, Ruby, Rust, PHP, C#). Enforces: meaningful names with intent-revealing prefixes (is/has/should/can for booleans, verbs for functions, PascalCase nouns for classes, UPPER_SNAKE_CASE for constants); short single-responsibility functions (≤20 lines, ≤3 params target); comments explain WHY not WHAT, no commented-out code; explicit error types with context-rich logs (no secrets); zero magic literals; zero dead code; bounded file size (≤300 lines target); ordered import groups (stdlib → third-party → internal absolute → relative) with no circular deps."
---

# Clean Code Rules

> Auto-applied to every source file. Reviewers and agents MUST comply.

## Naming
- Names must be self-explanatory and readable as English. NO single-letter names except loop counters (`i`, `j`).
- Booleans: prefix with `is`, `has`, `should`, `can` (e.g. `isReady`, `hasPermission`).
- Functions: start with a verb (`getUser`, `calculateTotal`).
- Class / Type: PascalCase, noun.
- Constants: UPPER_SNAKE_CASE.
- Avoid abbreviations except established standards (URL, ID, API, HTTP).

## Functions
- ≤ 20 lines (target). If longer — split or document why.
- ≤ 3 parameters (target). More → use object / struct.
- Single Responsibility: one function = one job.
- NO hidden side effects (parameter mutation, global state writes).
- Pure when possible.

## Comments
- Code self-documents first. Comments explain **why**, not **what**.
- DO NOT comment-out code; delete it (git keeps history).
- Update / remove comments when related code changes.

## Error handling
- Catch errors at boundaries (controller / handler), NOT everywhere.
- Each error has a concrete code/type. NEVER `throw new Error("something went wrong")`.
- Log with full context (correlationId, userId, params), but NEVER log secrets.

## Magic values
- No magic numbers / strings. Extract into named constants.
- Use enums or const objects for fixed value sets.

## Dead code
- Remove unused imports, variables, functions.
- Remove expired feature flags.

## File size
- ≤ 300 lines (target). Larger → split into modules.
- One primary export per file (except types and barrel files).

## Imports
- Group order: stdlib → third-party → internal absolute → relative.
- NO circular imports.
