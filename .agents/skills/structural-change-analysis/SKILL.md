---
name: structural-change-analysis
description: "Skill for Stage 1–2 when a task involves refactoring, splitting views/controllers, deleting files, or parallel branch planning. Forces necessity check, duplicate detection, layer impact, ownership boundaries, and verifiable output shape before coding. Use with field-impact-analysis for schema tasks."
---

# Skill: Structural Change Analysis

## Goal

Before splitting files, deleting partials, or planning multi-dev branches, answer:

1. **What does done look like?** (concrete output / file tree / behavior)
2. **Do we need this now?** (necessity vs YAGNI)
3. **What breaks or moves?** (layer impact, blast radius)
4. **What duplicates?** (logic, markup, queries, locales)
5. **Who owns what?** (parallel work without merge wars)

Primary outcomes:
- Reject refactor-for-refactor's-sake.
- Catch "delete X" claims that are still referenced elsewhere.
- Split work at **ownership boundaries**, not arbitrary folders.
- Produce a team-ready decision table + PR sequence.

## Use when

- "Tách index / filter / partial cho từng type"
- "Xóa file không dùng nữa"
- "3 dev làm song song — branch thế nào?"
- "Có nên move folder / extract shared component?"
- UI hub with `if/elsif` branches growing across modules
- Before `/grooming` or `/write-spec` on refactor-heavy tasks

## Do NOT use alone when

- Task is only adding DB columns → use `field-impact-analysis` (or both).
- Task is code review only → use `code-review` skill.

## Analysis flow (mandatory order)

### 1) Inventory — what exists today

List **every file/symbol** the proposal touches. For each item record:

| Item | Referenced by (grep) | Runtime path | Owner domain |
|------|----------------------|--------------|--------------|
| `_foo.html.erb` | `index`, `house_series/index` | GET /properties | shared hub |

**Rule:** Never mark "unused" without `grep` across repo (views, controllers, turbo streams, specs, locales, Stimulus).

### 2) Necessity gate — "Có cần làm không?"

For each proposed change, pick one:

| Verdict | Meaning | Action |
|---------|---------|--------|
| **NOW** | Blocks parallel work, Figma, or spec | Include in scaffold PR |
| **LATER** | Nice structure, zero user value today | Defer; log in Decision Log |
| **SKIP** | No evidence / still referenced / YAGNI | Do not do |

Ask explicitly:
- Does Figma / spec require this?
- Does it reduce conflict for ≥2 devs **this sprint**?
- Can we ship feature with a thinner change?

### 3) Output shape — "Xong thì trông thế nào?"

Describe **verifiable** end state (not vibes):

```text
Routes:
  GET /properties/lands → Lands::PropertiesController#index

Views:
  app/views/lands/properties/index.html.erb  (no external_land branch)

Behavior:
  - Legacy GET /properties?type=land&source=internal → 302 /properties/lands
  - house_series still renders properties/_tabs
```

Include: routes, controller actions, partial tree, turbo stream targets, spec files to update.

### 4) Duplicate & drift check

| Check | Question |
|-------|----------|
| Markup duplicate | Will 3 index templates copy the same shell? → extract `_list_shell` |
| Logic duplicate | Same filter/query in 3 controllers? → keep query object, split views only |
| Locale duplicate | Keys under `properties.*` vs `lands.*` — migrate or alias? |
| CSS duplicate | Shared vs per-domain stylesheet boundary |
| Spec duplicate | One request spec per route or shared examples? |

**Prefer:** split **views + permits + query params** first; split controllers when actions diverge.

### 5) Layer impact map

| Layer | Change | Risk | Rollback |
|-------|--------|------|----------|
| Routes | new scope `/properties/lands` | 🟡 | revert routes.rb |
| Controller | thin dispatcher vs dedicated | 🟡 | single revert |
| View | split index | 🟢 if ownership clear | per-partial |
| Helper | `properties_table_headers` if/elsif | 🔴 hotspot | dispatch to domain helpers |
| Turbo stream | target id changes | 🟠 | keep stable dom ids |
| Spec | path + body assertions | 🟡 | update with feature |

Tag overall: 🟢 / 🟡 / 🟠 / 🔴 (same as workflow impact levels).

### 6) Parallel ownership matrix

When ≥2 devs:

| Dev | Branch | May edit | Must NOT edit |
|-----|--------|----------|---------------|
| A | `feat/...-land-internal` | `lands/properties/*`, `Lands::*` | `_house*`, external partials |
| B | `feat/...-land-external` | `external_lands/*` | ... |

**Shared files rule:** Only scaffold PR author touches dispatchers (`_filters`, thin `index`, hub `_tabs`). Feature PRs touch domain files only.

### 7) PR sequence (fastest safe path)

Default pattern:

```
PR0  foundation already on main (e.g. house series)
PR1  scaffold: delete dead UI, split dispatchers, stable dom ids  (~1d, 1 person)
PR2–4  parallel feature PRs per domain
PR5  optional cleanup (move folders, rename) — never block features
```

## Hard rules

- **grep before delete** — "không dùng nữa" must show zero references or explicit OUT in spec.
- **No big-bang move** — split behavior first, move folders last.
- **Hub components stay shared** until spec says otherwise (tabs across house_series / house_options / properties).
- **One scaffold PR** for shared hotspots; never 3 devs editing the same dispatcher.
- **Stable turbo/dom ids** across scaffold — changing `properties-list-desktop` id breaks inline delete streams.
- If uncertainty on Figma → **Open Question**, not delete.

## Red flags (push back)

- Deleting navigation used by another hub page
- 3 copy-paste `index.html.erb` with 80% identical markup
- Splitting controllers before views are stable
- Parallel branches all editing `properties_helper.rb`
- Removing semantic/search without updating queries + export + specs in same PR

## Output template (copy for Slack / grooming / spec)

```markdown
## Structural Change Analysis — <task name>

### Proposals reviewed
| Proposal | Verdict (NOW/LATER/SKIP) | Reason |
|----------|--------------------------|--------|
| Delete `_summary_card` | NOW | Figma removed; grep confirms only properties index |
| Delete `_tabs` | SKIP | Still rendered by house_series, house_options, properties |

### Target output (verifiable)
- Routes: ...
- Views: ...
- Behavior: ...

### Duplicate check
| Risk | Mitigation |
|------|------------|
| 3× index shell copy | Extract `properties/_index_shell` partial |

### Layer impact
| Layer | Change | Risk |
|-------|--------|------|
| View | ... | 🟡 |

### Ownership (parallel)
| Branch | Owner | Files |
|--------|-------|-------|

### PR sequence
1. PR1 scaffold — ...
2. PR2 land-internal — ...

### Open questions
1. ...
```

## Quick checklist (30-second pass)

- [ ] grep'd all delete candidates
- [ ] Stated what "done" looks like (routes + files + behavior)
- [ ] NOW/LATER/SKIP per proposal
- [ ] Duplicate markup/logic called out
- [ ] Shared hotspot identified → scaffold PR owner assigned
- [ ] Turbo/dom ids stable or migration listed
- [ ] Specs/locales in impact map

## Pairs with

| Skill | When |
|-------|------|
| `field-impact-analysis` | Schema / new columns / API fields |
| `karpathy-guidelines` | Keep scaffold minimal |
| `writing-ddd` | Lock routes + file tree in spec §Affected Components |
| `code-review` | Verify PR respects ownership matrix |
