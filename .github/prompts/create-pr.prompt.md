---
mode: agent
description: "Stage 9b of the Senior Workflow — Generate TRI-LINGUAL Pull Request artifacts. Outputs three copy-ready blocks: (1) PR Title in English (Conventional Commits, ≤72 chars); (2) Squash commit message in English with body; (3) PR Description in three collapsible language blocks (EN canonical, VI mirror, JP mirror), each containing Why / What / Impact / How to test / Spec links / Checklist / Rollback plan / Review focus. Refuses to run if /recheck-release is not READY. References the FINAL spec and recent commit history."
---

You are at **Stage 9b: Create PR**.

> **VI**: Bước 9b — Tạo Pull Request 3 ngôn ngữ (EN/VI/JP). Chỉ chạy khi `/recheck-release` đã READY.

## Precondition / Điều kiện
- `/recheck-release` returned `READY ✅`. If not → STOP, ask user to run it.
- **VI**: `/recheck-release` phải trả về READY. Nếu chưa → yêu cầu chạy lại.

## Project profile

If **ai-housemaker** (auto-detect or `(ai-housemaker)` flag) → follow `common/profiles/ai-housemaker.md` §9b for **Japanese PR body** + draft path. Title and squash commit remain English.

## Task

Produce **3 copy-ready blocks**:
1. **PR Title** — English (Conventional Commits)
2. **Squash commit message** — English with body
3. **PR Description** — 3 collapsible blocks: 🇬🇧 EN (canonical) / 🇻🇳 VI / 🇯🇵 JP

### Legacy format override (user preference)
- If the requester explicitly asks to keep their old PR description format,
   use that legacy structure instead of the tri-lingual collapsible template.
- In this case, still output blocks (1) Title and (2) Squash commit message,
   and output block (3) as a single legacy PR description.
- Legacy section order:
   - Summary
   - Background (Current behavior / Expected behavior)
   - Solution
   - Impact Analysis (Affected / Not affected)
   - Risks
   - Testing (Manual / Regression)
   - Main Files
- Keep Conventional Commit title rules unchanged.

## Steps

1. Get commit summary: `git log <base>..HEAD --oneline` and `git diff --stat <base>...HEAD`
2. Read FINAL spec in `docs/specs/` and `docs/ddd/`.
3. **If ai-housemaker profile** → follow `common/profiles/ai-housemaker.md` §9b + `ai-housemaker-pr-description` skill + `common/snippets/pr-description.ai-housemaker.ja.md`. **Stop** — do not use tri-lingual template below.
4. **Default (tri-lingual):** Read `.agents/skills/pr-conventions/SKILL.md` and fill `common/snippets/pr-description.trilingual.md`.
5. Output three copy-ready blocks: `### 1. Title`, `### 2. Commit msg`, `### 3. Description`.

Title and squash commit rules: **pr-conventions skill** (English Conventional Commits, ≤72 chars).

## ⚠️ Hard rules
- DO NOT fabricate spec links — verify they exist before linking.
- **NEVER** run `git push` — user pushes when ready. Generate PR text only.
- Title ≤ 72 chars; body wraps at 100 chars.
- For impact 🟠 / 🔴 — Rollback plan MUST have concrete steps, NOT just "revert PR".
- If PR > 500 lines diff → warn "consider splitting" but still output the PR.
- VI and JP blocks must mirror the EN content; do not omit sections.
- Exception: if legacy format override is explicitly requested by the user,
  skip tri-lingual block requirements and use the requester-preferred structure.
- If unresolved `🟡 Medium` findings exist, include `Deferred Medium Findings` with ticket/owner/target.
- Never allow deferred `🟡` for auth/authz, tenant scoping, payment, migration safety, or data integrity risks.

## 📤 Output format
3 separate blocks with code fences (```), with headers `### 1. Title`, `### 2. Commit msg`, `### 3. Description` so the user can copy each quickly.
