---
mode: agent
description: "Stage 3 of the Senior Workflow — Author Basic Design (BD) and Detail Design Document (DDD) as two separate Markdown files in docs/specs/ and docs/ddd/. Each file is TRI-LINGUAL: every section duplicated for EN (canonical) / VI / JP using collapsible <details> blocks. MUST include processing flow diagrams: BD has 1 high-level flowchart; DDD has 1 logical flowchart + sequence diagrams (happy + error path) + ERD if schema changes. Status starts as DRAFT — never set FINAL here (that is /check-spec). All decisions must include rationale. Reads writing-bd and writing-ddd skills first."
---

You are at **Stage 3: Write Spec**.

> **VI**: Bước 3 — Viết spec (Basic Design + Detail Design Document), xuất ra 2 file Markdown song ngữ EN/VI/JP.

## Preconditions / Điều kiện tiên quyết
- `/analyze-task` (Stage 1) and `/grooming` (Stage 2) outputs exist.
- All Open Questions are resolved. / Tất cả câu hỏi mở đã được giải quyết.
- If not → STOP, ask user to complete the previous stages.

## Task / Nhiệm vụ

Generate **two files** from templates:

**VI**: Tạo 2 file từ template:

1. **Basic Design (Thiết kế cơ bản)** → `docs/specs/<YYYY-MM-DD>-<kebab-slug>.md`
   - Use `docs/specs/_TEMPLATE.md`.
   - Focus on *what* and *why*. / Tập trung vào *cái gì* và *tại sao*.
2. **Detail Design Document (Thiết kế chi tiết)** → `docs/ddd/<YYYY-MM-DD>-<kebab-slug>.md`
   - Use `docs/ddd/_TEMPLATE.md`.
   - Focus on *how*: API contract, data model, flow + sequence diagrams, algorithms, error handling, test plan, rollout plan.
   - **VI**: Tập trung vào *làm thế nào*: API, data model, flow diagram, thuật toán, xử lý lỗi, test plan.

## 🌐 Tri-lingual rule / Quy tắc 3 ngôn ngữ (MANDATORY / BẮt buộc)

Every textual section in both files MUST appear in **3 languages**, in this order with collapsible blocks:

```markdown
### Section title (EN canonical text)

<details><summary>🇻🇳 Tiếng Việt</summary>

VI translation here.

</details>

<details><summary>🇯🇵 日本語</summary>

JP translation here.

</details>
```

- EN is canonical — VI and JP are translations of the EN text.
- Code blocks, tables, diagrams are NOT duplicated (they are language-neutral).
- API field names, error codes, type names stay in English everywhere.

## 🎨 Diagrams / Sơ đồ (MANDATORY / BẮt buộc)

### Basic Design must include / BD phải có
- **1 high-level flowchart** (Mermaid `flowchart TD` or `flowchart LR`) showing the end-to-end processing flow at component/actor level.
- **VI**: 1 flowchart tổng quan hiển thị luồng xử lý end-to-end.

### Detail Design Document must include / DDD phải có
- **1 logical flowchart** (Mermaid `flowchart`) showing the algorithm / decision branches.
- **≥1 sequence diagram for the happy path** (Mermaid `sequenceDiagram`).
- **≥1 sequence diagram for an error/edge path**.
- **1 ERD** (Mermaid `erDiagram`) IF data model changes.
- **VI**: 1 flowchart logic, ≥1 sequence happy path, ≥1 sequence error path, 1 ERD (nếu đổi schema).

## Must read first
- `.agents/skills/writing-bd/SKILL.md`
- `.agents/skills/writing-ddd/SKILL.md`
- `.cursor/rules/architecture.mdc` (so the spec doesn't violate existing architecture)

## Spec writing rules

1. **Default Status**: `DRAFT`. Never set `FINAL` here.
2. **Every decision** has a rationale (`Decision Log` section).
3. **API contract**: include request/response examples + all error codes.
4. **Data model changes**: migration plan + rollback plan.
5. **Test plan** is quantitative: # tests per layer, coverage target.
6. **Rollout plan**: feature flag? canary %? rollback steps?

## ⚠️ Hard rules
- DO NOT copy task description verbatim — synthesize.
- DO NOT assume — record assumptions explicitly in `Assumptions`.
- If information is missing → return to Stage 2, do not guess.
- **DO NOT run `git commit`** — write/edit spec files only (working tree). See `.cursor/rules/git-commit-policy.mdc`.
- **DO NOT** suggest or run `git add` + `git commit` for BD/DDD output. Commits start only after `/start-coding` + `/review-staged` READY.

## ➡️ Next step
Run `/recheck-spec` to self-audit and score.
