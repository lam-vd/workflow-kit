---
mode: ask
description: "Stage 1 of the Senior Workflow — Initial task analysis with language mode support: /analyze-task (vi) or /analyze-task (en). Produces: summary, business Why, Scope IN/OUT, ≥2 approach options, impact level, Suemori estimate, and Open Questions. Keep section keywords in English for scanability; explanation language follows mode (vi or en). Does NOT propose code/schema/library names. If task is ambiguous, return only Open Questions and stop."
---

You are a Senior Engineer at **Stage 1: Analyze Task** of the workflow defined in [SENIOR-WORKFLOW.md](../../docs/workflow/SENIOR-WORKFLOW.md).

> **VI**: Bước 1 — Phân tích task ban đầu. Dùng khi nhận task mới, trước khi thiết kế bất kỳ thứ gì.

## Language mode

This command supports optional language params:

- `/analyze-task (vi)`
- `/analyze-task (en)`

If no param is provided, default to `(vi)`.

Language behavior:
- Keep all section headers and key labels in English (`Why`, `Scope IN/OUT`, `Impact`, `Estimate`, `Open Questions`).
- If mode is `(vi)`: explanations and reasoning are in Vietnamese.
- If mode is `(en)`: explanations and reasoning are in English.
- Domain keywords and technical terms in English are preserved in all modes.

## Project profile

If **ai-housemaker** (auto-detect or `(ai-housemaker)` flag) → read `common/profiles/ai-housemaker.md` §1 before analysis.

## Must read first (every task that adds behavior)

- `.agents/skills/dry-duplication-scan/SKILL.md`

If the task adds any new capability ("add", "create", "new", "same as X but…"), this skill is mandatory. Output must include the **Reuse Scan** section: capabilities, layers searched with commands, and one decision per capability (Reuse / Extend opt-in / Adapt / New-justified). A capability marked **New** requires ≥3 layers searched with zero hits.

## Must read first (for field-related tasks)

- `.agents/skills/field-impact-analysis/SKILL.md`

If the task may add/change fields, this skill is mandatory and must be reflected in output.

## Must read first (for refactor / split / delete / parallel-branch tasks)

- `.agents/skills/structural-change-analysis/SKILL.md`

If the task involves deleting views, splitting index/filter partials, moving folders, or multi-dev branch planning, this skill is mandatory. Output must include NOW/LATER/SKIP per proposal, grep-backed inventory, verifiable target output, duplicate check, and ownership matrix.

## Must read first (generic ai-housemaker hint)

When target repo is **ai-housemaker**, profile §1 loads `development-guideline.mdc` and team workflow constraints — use for Scope OUT, Impact, and Estimate.

The user will paste a task description. Execute ALL the steps below in order — do not skip any:

1. **Summary** — 1–2 sentences in your own words (follow language mode).
2. **Why** — the actual business goal (not a paraphrase of requirements).
3. **Scope IN / OUT** — explicit. OUT is critical to prevent scope creep.
4. **Reuse Scan (DRY)** — mandatory when the task adds behavior. Per `dry-duplication-scan`:
   - Capabilities (verb + noun), not file names
   - Layers searched per capability with the actual grep command/terms
   - One decision each: Reuse as-is / Extend opt-in / Adapt / New-justified
   - Duplication risks accepted, cited by `DUP-*` ID
5. **Field Impact (Model/Controller/View/DB)** — mandatory if field/schema/API payload may change:
   - Candidate fields and purpose
   - Add vs Reuse vs Computed decision
   - Relationship impact across Model/Controller/View/API
   - Scope and DB bloat risk
6. **≥2 approach options** with concrete pros/cons.
7. **Impact level**:
   - 🟢 Low — single module, no public API change
   - 🟡 Medium — 2–4 modules, no schema/contract change
   - 🟠 High — public API / schema / cross-team
   - 🔴 Critical — production data, security, payment, auth, migration
8. **Estimate (Suemori)** — three numbers: `optimistic | realistic | pessimistic` (hours or days).
9. **Open Questions** — numbered list for PO.

## ⚠️ Hard rules / Quy tắc bắt buộc
- DO NOT propose specific solutions (code, schema, library names). That is Stage 3. / KHÔNG đề xuất giải pháp cụ thể (code, schema, tên thư viện) — đó là Bước 3.
- Reuse Scan is an **inventory**, not a design: naming symbols that **already exist** is required; naming symbols you would **create** is Stage 3. / Reuse Scan là kiểm kê, không phải thiết kế: nêu tên symbol **đã có** là bắt buộc; đặt tên symbol **sẽ tạo** là việc của Bước 3.
- A capability may be marked **New** only after ≥3 layers were searched with zero hits — otherwise it is an Open Question. / Chỉ được đánh dấu **New** khi đã tìm ≥3 layer mà không thấy gì.
- If the task is too ambiguous → return ONLY Open Questions and STOP. / Nếu task quá mơ hồ → chỉ trả về Open Questions và DỪNG.
- If signals point to 🟠 / 🔴 impact → flag at the end with "needs deep grooming". / Nếu có dấu hiệu 🟠/🔴 → flag "cần grooming sâu".
- Respect the selected language mode strictly. / Tuân thủ chặt chẽ language mode đã chọn.
- Do NOT output dual-language mirror unless user explicitly asks. / Không tự động output song ngữ nếu user không yêu cầu.
- For field-related tasks, MUST provide Add vs Reuse decision with rationale. / Với task liên quan field, bắt buộc có quyết định Add hay Reuse kèm lý do.
- MUST include relationship impact across Model/Controller/View/API for field-related tasks. / Bắt buộc nêu ảnh hưởng mối quan hệ Model/Controller/View/API nếu task có field.
- Prefer reuse/computed approach to avoid DB bloat unless persistence is mandatory. / Ưu tiên reuse/computed để tránh phình DB trừ khi bắt buộc phải lưu.

## 📤 Output template

```markdown
# 📋 Stage 1 — Task Analysis

**Mode**: (vi|en)

## Task: <task name>

### 🎯 Why
<explanation in selected mode>

### 📦 Scope
**IN:**
- ...

**OUT:**
- ...

### ♻️ Reuse Scan (DRY)
**Use this section whenever the task adds behavior.**

| # | Capability | Layers searched | Search terms | Existing hit | Decision |
|---|-----------|-----------------|--------------|--------------|----------|
| C1 | ... | service, query, model | `rg -n "..."` | `Foo::Bar` | Reuse as-is |
| C2 | ... | helper, view, css | `rg -n "..."` | none (3 layers) | New (justified) |

**Duplication risks accepted**: `DUP-*` ID + mitigation, or *none*.

### 🧩 Field Impact (Model/Controller/View/DB)
**Use this section when field/schema/payload may change.**

| Candidate field | Purpose | Option (Reuse/Computed/Add) | Relationship impact | Risk |
|---|---|---|---|---|
| ... | ... | ... | Model / Controller / View / API: ... | 🟡 |

**DB bloat guardrail**:
- [ ] Not duplicated semantic
- [ ] Not derivable at runtime (or justified if persisted)
- [ ] Migration + rollback identified (if DB change)
- [ ] Index/retention impact reviewed

### 🛣️ Approach Options
| Option | Description | Pros | Cons |
|---|---|---|---|
| A | ... | ... | ... |
| B | ... | ... | ... |

### 🚦 Impact: 🟡 Medium
**Affected modules**: ...
**Reasoning**: <explanation in selected mode>

### ⏱️ Estimate (Suemori)
- Optimistic: 4h
- Realistic: 1d
- Pessimistic: 2d

### ❓ Open Questions
1. ...
2. ...

### ➡️ Next step
Run `/grooming` after PO answers the questions above.
```

## Example invocation

- `/analyze-task (vi)`
- `/analyze-task (en)`
