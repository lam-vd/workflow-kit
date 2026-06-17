---
applyTo: "**"
description: "Architecture and layering rules applied repo-wide. Enforces clean-architecture style layering (UI/Controller → Application/Use Case → Domain → Infrastructure) with one-way dependencies through interfaces. Domain layer must remain framework/HTTP/DB-agnostic. Infrastructure implements domain-defined interfaces. Module boundaries require explicit public APIs (no cross-module deep imports), zero circular dependencies, constructor-based DI (no `new`-ing infrastructure inside domain), side effects pushed to system edges, and config validated at startup. Logging requires correlation IDs across requests, with metrics/traces on every external call. Flags violations at 🟠+ severity (domain importing HTTP/SQL, fat controllers, mutable global singletons)."
---

# Architecture Rules

## Layering (dependency direction)

```
UI / Controller / Handler
        ↓
   Application / Use Case
        ↓
       Domain
        ↓
  Infrastructure (DB, HTTP, FS)
```

- Each layer depends ONLY on the layer immediately below (via interfaces).
- Domain knows NOTHING about DB, HTTP, framework.
- Infrastructure implements interfaces declared by Application / Domain.

## Module boundaries
- Each module / feature exposes a clear public API (e.g. `index.ts`, `__init__.py`).
- NO cross-module shortcut imports through internal paths.
- NO circular dependencies between modules.

## Dependency Injection
- Inject dependencies via constructor / function params. DO NOT `new` infrastructure classes inside domain code.
- Tests must be able to mock dependencies.

## Side effects
- Side effects (I/O, network, time, randomness) live at the **edges** of the system.
- Core business logic is pure when possible.

## Configuration
- Config from env vars or config file. NEVER hardcoded.
- Validate config at startup; fail fast on missing/invalid values.

## Error boundaries
- Each entry point (HTTP route, message handler, scheduled job) has its own error boundary.
- Map domain errors → transport errors (HTTP status, message code) at the boundary.

## Logging & observability
- Log at use-case entry/exit + on error.
- Every request carries a correlation ID end-to-end.
- Metrics: counter (events) + histogram (latency) for every external call.

## Record lookup & authorization (IDOR prevention)
- NEVER load records by untrusted user-supplied ID (e.g. `Model.find(params[:id])`).
- Always scope lookups through the owning association or context (e.g. `@reservation.reservation_route` instead of `ReservationRoute.find(params[:id])`).
- Validate authorization / status guards BEFORE touching the database, not after loading.
- This applies equally to service objects, not just controllers.

## Resource cleanup (UI overlays & map objects)
- Any object placed on a shared canvas/map/DOM MUST store its reference for later cleanup.
- Factory/helper functions should only create what the caller needs — do NOT silently create extra objects that get discarded.
- When resetting UI state, tear down ALL created objects (renderers, polylines, markers, listeners).

## Violations to flag (severity 🟠+)
- Domain layer importing HTTP / SQL / framework code.
- Controller containing complex business logic (>10 lines of if/else).
- Multiple entry points hitting the DB directly without going through repositories.
- Mutable global state via singletons.
- `Model.find(params[:id])` without scoping through owning association (IDOR 🔴).
- UI overlay objects (markers, polylines, renderers) created without storing cleanup references.
