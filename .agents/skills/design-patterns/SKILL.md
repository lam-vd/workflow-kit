---
name: design-patterns
description: "Cheat-sheet of common design patterns to consult during Stage 7 implementation. Core principle: patterns serve problems, not the reverse — never apply a pattern without an articulated pain point. Covers when to use (and when to AVOID) Repository, Strategy, Factory, Adapter, Decorator/Middleware, Result/Either types, Saga/Outbox, CQRS — with concrete triggers and YAGNI guardrails. Lists frequently-abused patterns to be wary of (Singleton, God Service, Anemic Domain Model, premature abstraction). Includes a 5-question pre-pattern checklist and 5 refactor signals indicating when patterns become justified (duplicated logic ≥3 places, switch-on-type repetition, constructor params >5, class >500 lines, tests requiring 5+ mocks)."
---

# Skill: Design Patterns Cheat Sheet

## Core principles
- **Patterns serve problems, not the reverse.** Don't apply a pattern when there is no pain point.
- **YAGNI**: until you actually need an extension point — write code straight.
- **Prefer composition over inheritance.**
- **Prefer pure functions over classes** when state is not needed.

## Useful patterns

### Repository
- **When**: separate persistence from domain.
- **Shape**: interface in domain, implementation in infrastructure.
- **Note**: one repository per aggregate, NOT per table.

### Strategy
- **When**: multiple ways to perform one task, chosen at runtime.
- **Avoid**: creating a Strategy when there is only one implementation.

### Factory
- **When**: complex object construction needs encapsulation.
- **Avoid**: factory for trivial objects.

### Adapter
- **When**: integrating an API/lib whose shape doesn't match domain.
- **Benefit**: vendor-swap insulation.

### Decorator / Middleware
- **When**: cross-cutting concerns (logging, auth, retry, cache).
- **Note**: order matters — document it.

### Result / Either type
- **When**: replacing thrown exceptions for predictable errors.
- **Per language**: Rust `Result`, TS `Result<T,E>` or `neverthrow`, Go `(T, error)`.

### Saga / Outbox
- **When**: distributed transaction, eventual consistency.
- **Warning**: complex — only when truly needed.

### CQRS
- **When**: read-heavy + read shape ≠ write shape.
- **Warning**: doubles code; apply only to bounded contexts with clear business case.

## Frequently abused patterns (be careful)
- **Singleton**: causes hidden coupling, hard to test. Replace with DI.
- **God Service**: one service doing 20 things. Split by use case.
- **Anemic Domain Model**: entities with only getters/setters, all logic in services. Push behavior into entities.
- **Premature abstraction**: interface with exactly one implementation.

## Pre-pattern checklist
- [ ] What is the specific problem? (1 sentence)
- [ ] How does current code hurt? Measurable?
- [ ] Does this pattern solve THAT problem?
- [ ] Is there a simpler alternative?
- [ ] Is cost/benefit positive over the next 6 months?

## Refactor signals (when a pattern becomes justified)
- Same logic copy-pasted in ≥3 places → consolidate.
- Repeated switch/if-else on type → polymorphism / strategy.
- Constructor with >5 params → builder or factory.
- Class >500 lines → split by responsibility.
- Tests requiring 5+ mocks → split modules.
