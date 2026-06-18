---
name: field-impact-analysis
description: "Skill for Stage 1 task analysis when a request may add/change fields. Evaluates model/controller/view relationship, data ownership, lifecycle, and blast radius before proposing schema growth. Prevents DB bloat and cross-layer regressions by forcing add-vs-reuse decisions, impact mapping, and guardrails."
---

# Skill: Field Impact Analysis (Model/Controller/View/DB)

## Goal

Before any design/coding, determine whether a new field is truly needed, where it belongs, and what it impacts.

Primary outcomes:
- Avoid unnecessary schema growth.
- Avoid hidden regressions across Model/Controller/View/API.
- Keep ownership and source-of-truth clear.

## Use when

- Task mentions new attribute/column/flag/status/metadata.
- UI asks to display/store extra info.
- API contract asks for new request/response fields.
- Existing behavior seems to need "just one more field".

## Analysis flow

### 1) Clarify field intent

For each candidate field, answer:
- Business purpose: what decision/behavior requires this field?
- Data type and cardinality.
- Lifecycle: created/updated/expired/deleted when?
- Sensitivity: PII/financial/security related?

If purpose is unclear, do not add field; raise open question.

### 2) Add vs Reuse decision

Decide one option explicitly:
- Reuse existing field/model.
- Derive value at runtime (computed), no persistence.
- Add new field (only if required).

Rule:
- Prefer reuse/computed over schema growth unless persistence is mandatory.

### 3) Relationship map (Model/Controller/View)

Create an impact matrix per candidate field:

| Layer | Impact questions |
|---|---|
| Model / DB | Which table? nullable? default? index needed? migration/rollback? |
| Controller / Service | Which params/permit/validation/use-case path changes? |
| View / UI | Which forms, hidden fields, labels, chips, display logic? |
| API / Contract | Request/response changes? backward compatibility? |
| Jobs / Integrations | Background jobs, webhooks, exports/imports affected? |

### 4) Blast radius and risk

Classify scope:
- Local (single module)
- Cross-module
- Contract/schema impact

And risk:
- Data integrity risk
- Backward compatibility risk
- Operational risk (migration, rollback, deploy order)

### 5) DB bloat guardrails

Before approving new field, verify:
- Not duplicate of existing semantics.
- Not temporary/debug-only data.
- Not storing derivable data unless performance requires it.
- Storage size growth is acceptable.
- Query/index impact understood.
- Retention policy exists for large text/json fields.

### 6) Best-practice recommendation

Output must include:
- Recommended approach (Reuse / Computed / Add field).
- Why this is best practice.
- If add field: required safeguards (validation, migration, rollback, docs, tests).
- Explicit out-of-scope items to prevent accidental expansion.

## Hard rules

- Never add field by default convenience.
- Never add field without ownership and lifecycle.
- Never ignore migration rollback path.
- Never skip View/UI and API impact checks.
- If uncertainty is high, block and ask clarifying questions first.

## Output mini-template

```markdown
## Field Impact (Model/Controller/View/DB)

### Candidate fields
| Field | Purpose | Option (Reuse/Computed/Add) | Decision reason |
|---|---|---|---|
| <name> | <why> | <option> | <reason> |

### Relationship impact map
| Layer | Impact | Risk |
|---|---|---|
| Model/DB | ... | 🟡 |
| Controller/Service | ... | 🟢 |
| View/UI | ... | 🟡 |
| API/Contract | ... | 🟠 |

### DB bloat guardrail check
- [x] Not duplicated semantic
- [x] Not derivable at runtime (or justified if persisted)
- [x] Migration + rollback plan identified
- [x] Index/retention impact reviewed

### Recommendation
- Decision: <Reuse/Computed/Add>
- Scope impact: <low/medium/high>
- Best-practice note: ...
```
