---
name: dry-duplication-scan
description: "Pre-implementation DRY gate — before generating ANY new code from a task, scan the codebase layer by layer (route / controller / service / query / model / validation / helper / view / JS / CSS / locale / config / spec) for logic that already exists. Produces a grep-backed Scan evidence table + Reuse decision per capability (Reuse as-is / Extend opt-in / Adapt / New-justified) + duplication smell IDs (DUP-*). Use at Stage 1 (analyze-task) to size scope and at Stage 7 (start-coding) before writing code; also usable at Stage 8 to catch duplication introduced by the diff."
---

# Skill: DRY / Duplication Scan (pre-implementation)

## Goal

Stop the most expensive LLM failure mode: **writing code that already exists somewhere else in the repo**.

The rule is not "avoid duplication" (too vague). The rule is:

> **No new symbol enters the codebase without a recorded search that failed to find an existing one.**

Output is evidence, not a promise: a table of what was searched, where, and what was found.

## Use when

- Stage 1 `/analyze-task` — before estimating or picking an approach.
- Stage 7 `/start-coding` — in the implementation plan, before writing code.
- Any time the task says "add", "create", "new", "also support", "same as X but…".
- Stage 8 `/review-staged` — reverse direction: does the diff duplicate existing code?

## Do NOT use alone when

- Task deletes / splits / moves files → `structural-change-analysis` (duplication *created by* the split).
- Task adds DB columns → `field-impact-analysis` (reuse vs computed vs add for **data**; this skill covers **logic**).
- Task edits code with ≥2 consumers → `.cursor/rules/shared-abstraction-safety.mdc` (blast radius of reuse).

---

## Phase 1 — Capability list (what the task actually asks for)

Decompose the task into **capabilities**, not files. A capability is one verb + one noun.

| # | Capability | Layer it would live in |
|---|------------|------------------------|
| C1 | Filter customers by status | query / scope |
| C2 | Format JPY amount for display | helper / presenter |
| C3 | Notify staff on approval | service / job |

Rule: if you cannot name the capability, you cannot search for it — return an Open Question instead.

Search terms per capability = **synonyms**, not the name you were about to use:
- verb synonyms: `create/build/make/generate/register`, `fetch/get/find/load/lookup`, `send/notify/deliver/dispatch`
- noun synonyms + domain JA/EN pairs: `customer/顧客/client`, `property/物件/bukken`
- the **outcome**, not the mechanism: search `deadline` before writing `calculate_due_date`

---

## Phase 2 — Per-layer search matrix (mandatory)

For each capability, grep **every layer that could already own it**. Do not stop at the first layer.

| Layer | Search here | Looking for |
|-------|-------------|-------------|
| Route / endpoint | `config/routes.rb`, API route files | An action that already exposes this |
| Controller / action | `app/controllers/**`, `concerns/**` | Same params permit, same before_action, same flow |
| Service / use-case / job | `app/services/**`, `app/jobs/**`, `app/commands/**` | A `.call` that already does this orchestration |
| Query / scope | model `scope`, `app/queries/**` | Same WHERE / ORDER / JOIN combination |
| Model / domain method | `app/models/**` | Instance/class method computing the same value |
| Validation | model validations, form objects, DB constraints | Same rule enforced at another layer already |
| Helper / presenter / decorator | `app/helpers/**`, `app/presenters/**`, `app/decorators/**` | Same formatting / label / badge logic |
| View / partial / component | `app/views/**`, `app/components/**` | A partial rendering the same block |
| JS / Stimulus | `app/javascript/controllers/**` | A controller with the same DOM behavior |
| CSS / design token | `app/assets/stylesheets/**`, token files | An existing BEM block or utility |
| Locale / i18n | `config/locales/**` | An existing key with the same text |
| Config / constant / enum | initializers, `enum`, constants, ENV | A list/state set already declared |
| Spec / shared example | `spec/**`, `spec/support/**` | Shared examples / factories to reuse |

**Evidence rule:** record the actual search, not "I checked".

```
rg -n "due_date|deadline|期限" app/models app/services app/helpers
```

**Stop rule:** ≥3 layers searched and zero hits ⇒ genuinely new. <3 layers searched ⇒ scan incomplete, do not write code.

---

## Phase 3 — Reuse decision (one per capability)

| Decision | When | Cost of getting it wrong |
|----------|------|--------------------------|
| **Reuse as-is** | Existing symbol matches intent and signature | — |
| **Extend (opt-in)** | Existing symbol is 90% right; add a param/flag defaulting to current behavior | 🟠 if default changes → cross-screen regression |
| **Adapt (new, similar)** | Same shape, genuinely different domain rules; copying is honest | 🟡 drift — note the sibling in the plan |
| **New (justified)** | ≥3 layers searched, no hit | 🟢 |

Preference order: **Reuse > Extend opt-in > Adapt > New**.

Never choose **Extend** on a shared symbol without reading `.cursor/rules/shared-abstraction-safety.mdc` and listing consumers.

---

## Phase 4 — Duplication smells (cite these IDs in findings)

| ID | Smell | Why it hurts |
|----|-------|--------------|
| `DUP-NAME-01` | Two symbols, same behavior, different names (`fetch_x` / `get_x`) | Bug fixed in one, not the other |
| `DUP-WRAP-01` | New helper wraps a one-line model/AR method | Indirection with zero value |
| `DUP-DELEG-01` | New service delegates entirely to an existing one | Extra layer, same logic |
| `DUP-QUERY-01` | Same WHERE/scope inlined in 2+ controllers | Scope drift, N+1 hidden in one copy |
| `DUP-QUERY-02` | New `*Query` service = single relation filter + order, one consumer | Prefer model scope (`TenantUser.assignable_for_schedules`) — No.146 |
| `DUP-VIEW-01` | New partial ≈ existing partial with 1–2 differences | Design drift between screens |
| `DUP-I18N-01` | New locale key with text identical to an existing key | Two texts to update, one gets missed |
| `DUP-CSS-01` | New class re-declaring an existing BEM block / token value | Visual drift, dead CSS |
| `DUP-ENUM-01` | Status/type list re-declared in model + JS + locale + view | States added in one place only |
| `DUP-VALID-01` | Same rule in model, controller, JS and DB with different limits | Silent mismatch at the boundary → hand to `edge-case-boundary-review` |

---

## Output template

```markdown
## DRY Scan — <task name>

### Capabilities
| # | Capability | Target layer |
|---|-----------|--------------|
| C1 | ... | ... |

### Scan evidence
| # | Layers searched | Search command / terms | Hits |
|---|-----------------|------------------------|------|
| C1 | service, query, model | `rg -n "deadline\|期限" app/` | `Customer#due_at`, `Deadlines::Calculate` |
| C2 | helper, view, css | `rg -n "format_jpy\|円" app/helpers app/views` | none (3 layers, 0 hits) |

### Reuse decisions
| # | Decision | Target symbol | Rationale | Blast radius |
|---|----------|---------------|-----------|--------------|
| C1 | Reuse as-is | `Deadlines::Calculate` | Same rule | — |
| C2 | New (justified) | `format_jpy` helper | 3 layers, 0 hits | 🟢 |

### Duplication risks accepted
| ID | Item | Mitigation |
|----|------|-----------|
| `DUP-VIEW-01` | `_land_row` ≈ `_house_row` | Extract shared `_property_row` in follow-up — logged |

### New symbols introduced (must be empty or justified above)
- `app/helpers/currency_helper.rb#format_jpy`
```

---

## Hard rules

- **No new file, class, method, partial, Stimulus controller, CSS block, or locale key without a row in Scan evidence.**
- "I checked" without a command/term is **not** evidence.
- A hit that is 90% right → **Extend opt-in** or **Reuse**, never a silent parallel copy.
- Extending shared code → default behavior for existing callers must be unchanged (`shared-abstraction-safety.mdc`).
- Reuse decisions belong in the Stage 7 implementation plan **before** code is written, not in the PR description after.
- If a capability has no name, STOP → Open Question. Do not guess and implement.

## Quick checklist (30-second pass)

- [ ] Capabilities named (verb + noun)
- [ ] ≥3 layers searched per capability, with commands recorded
- [ ] Synonyms + JA terms used, not only the name I planned to use
- [ ] One explicit decision per capability (Reuse / Extend / Adapt / New)
- [ ] Every `Extend` on shared code has a consumer list
- [ ] Every `New` has ≥3 empty layers as proof
- [ ] Duplication smells cited by `DUP-*` ID

## Pairs with

| Skill / rule | When |
|--------------|------|
| `field-impact-analysis` | Duplication of **data** (columns) rather than logic |
| `structural-change-analysis` | Duplication **created by** splitting/moving files |
| `shared-abstraction-safety.mdc` | Chosen "Extend" on a symbol with ≥2 consumers |
| `edge-case-boundary-review` | `DUP-VALID-01` — same rule, different limits per layer |
| `code-review` | Stage 8 reverse scan: did the diff add a parallel copy? |
