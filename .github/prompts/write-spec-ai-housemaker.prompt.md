---
mode: agent
description: "Stage 3 variant for ai-housemaker (Rails 7.2 UI/feature). Authors BD (tri-lingual EN/VI/JP per workflow) + DDD in ai-housemaker best-practice format: scope, current state, gap, domain rationale, reuse table, HTTP error matrix, security, FR map. Also produces docs/ddd/<slug>.vi.md (Vietnamese, Notion-ready). Reads writing-bd, writing-ddd, rails-ui-layouts. Status DRAFT only — FINAL is /check-spec."
---

You are at **Stage 3: Write Spec — ai-housemaker**.

> **VI**: Viết spec cho repo **ai-housemaker** (Rails + Hotwire + Tailwind + Preline). BD giữ chuẩn workflow (EN/VI/JP). DDD theo format nội bộ (no.82 / best practices).

## Preconditions

- `/analyze-task` and `/grooming` done; Open Questions resolved.
- Inspect **actual codebase** before writing §2 Hiện trạng — cite real file paths.
- For UI tasks: read `.agents/skills/rails-ui-layouts/SKILL.md` in senior-workflow-kit (or ai-housemaker if copied).

## Output files (ai-housemaker repo)

| File | Language | Audience |
|---|---|---|
| `docs/specs/<YYYY-MM-DD>-<kebab-slug>.md` | Tri-lingual EN/VI/JP | Workflow / audit |
| `docs/ddd/<YYYY-MM-DD>-<kebab-slug>.md` | EN canonical | Implementation source of truth |
| `docs/ddd/<YYYY-MM-DD>-<kebab-slug>.vi.md` | **Vietnamese only** | Notion / team review |

Template DDD EN: `senior-workflow-kit/docs/ddd/_TEMPLATE-ai-housemaker.md`
Template DDD VI: `senior-workflow-kit/docs/ddd/_TEMPLATE-ai-housemaker.vi.md`

## BD rules (unchanged from `/write-spec`)

- Use `docs/specs/_TEMPLATE.md`.
- Tri-lingual collapsible blocks for textual sections.
- 1 high-level Mermaid flowchart.
- Status: **DRAFT** only.

## DDD rules — ai-housemaker format (EN canonical)

**Mandatory section order** (do not skip; use N/A with justification if empty):

1. **Scope & phase goal** — In / Out / one-sentence phase goal
2. **Current state** — table: component | path | current behavior
3. **Gap analysis** — checklist vs Figma/ticket
4. **Data model** — mapping table; migration yes/no + up/down if yes
5. **Domain / services** — service table; decisions with **Rationale** and **Won't do**
6. **Routes, controllers, forms** — route table, strong params, pseudocode for non-trivial branches
7. **Views & UI patterns** — file tree, wireframe text, **reuse table** (e.g. `customers/_form_modal`)
8. **HTTP responses & errors** — matrix per endpoint; edge-case table (≥5 rows for 🟡+ tasks)
9. **i18n** — namespace; JA user-facing keys (not hardcoded EN in UI)
10. **Security & tenant isolation** — auth, `policy_scope`, policy methods, cross-tenant prevention
11. **Observability** — AuditLog via services; metrics only if needed
12. **Diagrams** — 1 logical flowchart + ≥1 happy sequence + ≥1 error sequence; ERD if schema changes
13. **Test design** — T1–Tn table + breakdown by layer (request/form/mailer/policy)
14. **Implementation order & rollback**
15. **Requirement mapping** — Figma/PO/ticket row → DD § → test id

### Rails UI conventions (ai-housemaker)

- Dashboard features: `layouts/dashboard`, `dashboard_page_shell`, `ahm-card`, `ahm-modal-*`
- Modals: Preline `hs-overlay` + Turbo Frame (see `customers/_form_modal.html.erb`)
- Auth screens only: `layouts/auth`, `auth-entry` BEM — **not** for dashboard profile/settings
- Business logic: `*Manager::` services — controllers thin
- Locale default JA; i18n via `config/locales/*.yml`
- No automated test suite yet — specify **quantitative** request/form specs to add

### DDD Vietnamese file (`.vi.md`)

- Single language Vietnamese; **no** EN/JP collapsible blocks.
- **Prose format kiểu no.82** (reference: meeting_history DD): đoạn mở đầu task → In/Out scope → Phase goal; hiện trạng dạng bullet path + "Current behavior" + "Gap lớn nhất"; subsection `## **Tên section**` và `### **N. ...**`; mỗi quyết định domain có **Lý do** / **Không làm** inline.
- EN canonical DDD giữ structure §1–15 (tables + matrices); VI `.vi.md` là bản Notion đọc nhanh, **nội dung kỹ thuật phải khớp EN**.
- Same topical sections as EN DDD; tables and code blocks unchanged.
- Test section: **Coverage đã có** + **Coverage cần bổ sung** (không chỉ bảng T1–Tn).
- **Notion Mermaid:** bọc message trong `"..."`; tránh `;` `:` `?` `/` trong arrow label; participant alias PascalCase không khoảng trắng; không dùng `alt/else` nếu Notion báo lỗi — tách thành 2 message tuần tự.

## Must read first

- `senior-workflow-kit/.agents/skills/writing-bd/SKILL.md`
- `senior-workflow-kit/.agents/skills/writing-ddd/SKILL.md`
- `senior-workflow-kit/.agents/skills/rails-ui-layouts/SKILL.md` (UI tasks)
- `senior-workflow-kit/docs/ddd/_TEMPLATE-ai-housemaker.vi.md`

## Hard rules

- **DRAFT** only — never FINAL here.
- §2 Current state must reference **real paths** from repo grep/read.
- Every domain decision: **why** + what we explicitly **won't** change.
- Do not copy ticket text verbatim — synthesize.
- If FINAL spec exists, changes require new file `-v2` or new date slug — do not silently edit FINAL.
- **DO NOT run `git commit`** — see `.cursor/rules/git-commit-policy.mdc`.

## ➡️ Next step

Run `/recheck-spec` on BD + DDD (EN). Share `.vi.md` with team via Notion.
