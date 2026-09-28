---
name: feature-cutover-orphan-review
description: >-
  Stage 7–8 (and peer review) skill for partial feature ship / CRUD-UI removal /
  domain-kept cutovers. Greps call sites across app, jobs, policies, FE clients,
  locales, and constants; classifies Dead-sure / Ops-only / Soft-dead / Shared-keep.
  Also runs a short ship-contract sweep (authz path parity, client stop vs server
  phases, schema vs producer, dual env flags). Stack-agnostic — use for any
  backend/FE monorepo. Complements deadcode-ui-migration-review (UI screen replace).
---

# Skill: Feature Cutover & Orphan Review

## Goal

When a feature **ships incompletely** or **removes one surface while keeping another** (e.g. admin CRUD UI gone, domain records + search still used; new API added but no client; enqueue helper with zero web writers), leftovers become:

- **True orphans** (safe to delete or must wire),
- **Ops-only** paths (rake/CLI — keep, document),
- **Soft-dead** (locale keys, unused constants — follow-up),
- **Silent ship risks** (client stops waiting before server finishes; authz on transport A weaker than HTTP).

This skill forces a **call-site matrix** + a short **contract sweep** before merge.

## When to use

- PR removes CRUD UI / admin screens / FE flows but keeps models, jobs, or knowledge data
- PR adds controllers, jobs, enqueue helpers, Cable/WS channels, or FE URL templates
- Stage 7–8 cleanup after “split into another PR” / “ops will reembed later”
- Peer `/review-branch` when diff looks like upgrade/refactor + new async surface
- Parallel to `structural-change-analysis` (plan) and `deadcode-ui-migration-review` (full-page → modal UI only)

## Do NOT use alone when

- Pure full-page → modal/Turbo-frame UI replace → prefer `deadcode-ui-migration-review`
- Diff-only style / copy → `code-review` Lean gates
- Field/schema blast radius without cutover → `field-impact-analysis`

## Relationship

| Skill | Role |
|-------|------|
| `structural-change-analysis` | Stage 1–2: decide what to delete vs keep |
| `deadcode-ui-migration-review` | UI screen migration orphans (views/partials/frames) |
| `feature-cutover-orphan-review` | **This file** — non-UI + partial-ship orphans + contract sweep |
| `lean-facade-review` | Thin JSON getters / predicates / redundant I18n `default:` (often Soft-dead nits) |
| `shared-abstraction-safety` | Don’t put feature side effects into shared helpers |
| `integration-regression-review` | Cross-layer runtime flows once surfaces are wired |

---

## Hard rules

1. **Grep before “unused”** — zero call sites in `app/` (+ FE clients, Stimulus/JS, jobs that enqueue from web) before Dead-sure.
2. **Ops-only ≠ dead** — rake/CLI/ops task that calls enqueue is a **documented path**, not an orphan. Resolve “no app callers” only after confirming ops/seed path exists **or** Decision Log says manual-only.
3. **Domain keep ≠ CRUD keep** — model/association/policy may stay if another feature still reads the data; delete only surfaces with zero consumers.
4. **Unwired public endpoint is a finding** — route + controller with no FE/service caller → wire, hide, or delete in the same PR (attack surface / confusion).
5. **FE attribute set ≠ FE attribute read** — `data-*-url-value` / URL template in HTML that JS never reads → Soft-dead or unfinished wire-up.
6. **Never invent bugs** — comment only with grep evidence or a runnable repro. Skip speculative “might fail in prod” without a path in the diff.
7. **Pre-deploy / stub framing** — if the team has not deployed and data is stub-only, do **not** escalate missing prod backfill as 🔴 unless the DoD requires it.

---

## Classification matrix

| Class | Meaning | Default action |
|-------|---------|----------------|
| **Dead-sure** | 0 callers in app + FE + jobs triggered from web | Delete in this PR **or** 🟠 must if left as public surface |
| **Ops-only** | Only rake/CLI/seed/ops job | Keep; document in PR / Decision Log; optional rake spec |
| **Soft-dead** | Locale twin, unused constant, unused Stimulus target | 🔵 nit / follow-up prune |
| **Shared-keep** | Still used by another feature | Keep; do not delete in cutover PR |
| **Fallback-only** | Old path still reachable (redirect/gate incomplete) | Treat as live until gate is modal-only / removed |

Emit a table in the review (or paste-ready comment) with one row per candidate.

---

## Audit flow (mandatory order)

### 1) Name the cutover

One sentence:

> “Product keeps **A** (data/search/async); removes or never shipped **B** (CRUD UI / FE client / enqueue-from-web).”

### 2) Inventory candidates from the diff

For each new or touched symbol in:

| Layer | Examples |
|-------|----------|
| Policy / auth helper | Policy class only used by deleted controllers |
| Service / enqueue | `Enqueue*`, `*Job.perform_later` wrappers |
| Controller / route | `create`/`update` with no link/button/fetch |
| Realtime channel | subscribe/dismiss actions |
| FE client | store action, axios call, Stimulus value |
| Locale / constants | keys/enums only referenced by removed UI |
| Seed vs runtime | fixture calls sync refresh; app uses async job |

### 3) Grep call sites (evidence rule)

For each candidate, record:

| Symbol | `app/` hits | FE/JS hits | Spec hits | Ops/seed/rake | Class |
|--------|-------------|------------|-----------|---------------|-------|

**Specs alone do not keep a surface “product-live.”** Specs + zero product callers → still Dead-sure or Ops-only for the product path.

### 4) Ship-contract sweep (same PR class)

Run even when orphans look fine. Answer each with **PASS / FAIL / N/A + evidence**:

| ID | Check | FAIL signal |
|----|-------|-------------|
| **CONTRACT-AUTHZ-01** | Mutating realtime/transport path uses the **same** authorization as HTTP for that resource | Channel allows ID by membership only; HTTP uses policy_scope / tenant scope |
| **CONTRACT-ASYNC-01** | Client stop/polling/completed condition does not fire **before** a required server phase finishes | UI stops on `completed` while secondary job still running → empty panel |
| **CONTRACT-SCHEMA-01** | Producer output ⊆ consumer/schema allow-list (and vice versa for required fields) | Enum/schema allows labels the prompt/generator never emits; or UI expects a type never produced |
| **CONTRACT-ENV-01** | Live path documented when **≥2** flags/stubs must both be off/on | One stub false still hits SDK stub; docs mention only one flag |
| **CONTRACT-SEED-01** | Local seed path vs staging/prod refresh path are intentional | Seed sync OK; prod needs ops enqueue — documented, not “forgotten web caller” |
| **CONTRACT-DISMISS-01** | User dismiss/cancel has a defined success path when transport fails | Fire-and-forget WS only; HTTP ignored; `!ok` swallowed with no backup |

Tag findings `[cutover]` or `[contract]`.

### 5) Severity guide

| Finding | Typical severity |
|---------|------------------|
| Mutating action authz weaker on alternate transport | 🔴 / 🟠 `must` |
| Client completes UX before required server phase; empty/wrong UI | 🟠 `must` or 🟡 if workaround exists |
| Public controller/route with zero product callers | 🟠 `should` (wire/delete) |
| Enqueue helper with zero callers **and** no ops/rake/seed | 🟠 `should` |
| Enqueue helper ops/rake only (after verify) | 🔵 resolve / Decision Log |
| Schema/enum drift (extra unused labels) | 🟡 `suggestion` |
| Unused locale / Stimulus target / constant | 🔵 `nit` |
| Dual env flags undocumented | 🟡 `suggestion` |

---

## Stage 1–2 hook (with `structural-change-analysis`)

When planning “remove UI, keep data”:

1. List **keep** surfaces (readers: search, AI, reports).
2. List **remove** surfaces (CRUD controllers, policies-only-for-CRUD, FE pages).
3. For each remove candidate → grep → class in matrix above.
4. Decide **ops path** for regeneration (rake/job) **before** deleting web writers.
5. Log Ops-only + Soft-dead in Decision Log so Stage 8 does not re-litigate.

---

## Output snippet (for Full report or peer Lesson)

```markdown
### Cutover & contract (§)

| Symbol | Class | Evidence | Action |
|--------|-------|----------|--------|
| … | Ops-only | rake X | keep + document |
| … | Dead-sure | 0 app/FE callers | delete or wire |

Contract: AUTHZ-01 PASS/FAIL — …; ASYNC-01 …; SCHEMA-01 …
```

---

## Anti-patterns (reviewer)

| Don’t | Do |
|-------|----|
| “Delete all policies because CRUD UI is gone” | Grep readers; Shared-keep if search/AI still authorize via policy |
| “No `app/` caller → must delete enqueue” | Check rake/seed/Decision Log → Ops-only |
| Escalate missing prod backfill on stub/pre-deploy projects | Match team DoD; use 🟡/Decision Log |
| Invent IDOR without tracing both auth paths | Cite channel vs HTTP policy code |
| Flag seed sync embed as wrong because prod uses jobs | CONTRACT-SEED-01 intentional split |

---

## Quick grep patterns (adapt to stack)

```bash
# Symbol call sites (example)
rg -n "EnqueueFoo|FooPolicy|FoosController" app/ spec/ lib/ --glob '!**/enqueue_foo.rb'

# FE URL / template wiring
rg -n "fooUrlTemplate|foos#create|/foos" app/javascript src/ --glob '!**/node_modules/**'

# Locale key usage
rg -n "foos\.index\.unused_key" config/locales app/
```
