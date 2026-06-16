---
mode: agent
description: "Stage 9b of the Senior Workflow — Generate TRI-LINGUAL Pull Request artifacts. Outputs three copy-ready blocks: (1) PR Title in English (Conventional Commits, ≤72 chars); (2) Squash commit message in English with body; (3) PR Description in three collapsible language blocks (EN canonical, VI mirror, JP mirror), each containing Why / What / Impact / How to test / Spec links / Checklist / Rollback plan / Review focus. Refuses to run if /recheck-release is not READY. References the FINAL spec and recent commit history."
---

You are at **Stage 9b: Create PR**.

## Precondition
- `/recheck-release` returned `READY ✅`. If not → STOP, ask user to run it.

## Task

Produce **3 copy-ready blocks**:
1. **PR Title** — English (Conventional Commits)
2. **Squash commit message** — English with body
3. **PR Description** — 3 collapsible blocks: 🇬🇧 EN (canonical) / 🇻🇳 VI / 🇯🇵 JP

## Steps

1. Get commit summary:
   ```sh
   git log <base>..HEAD --oneline
   ```
2. Read relevant FINAL spec in `docs/specs/` and `docs/ddd/`.
3. Read `.agents/skills/pr-conventions/SKILL.md` for project conventions.
4. Generate the three blocks below.

## 1. PR Title

Format: `<type>(<scope>): <imperative summary, ≤72 chars>`

- types ∈ { feat, fix, refactor, perf, test, docs, build, ci, chore }
- imperative mood: "add", "remove", "update" (NOT "added", "adding")
- DO NOT end with a period
- DO NOT translate title to VI/JP — keep English (commit history searchability)

Example:
```
feat(scheduler): add per-platform field validation
```

## 2. Squash commit message

```
<PR title>

<body — 1 paragraph WHY + bullet list WHAT (English)>

Refs: <task-id or spec link>
```

## 3. PR Description (tri-lingual)

Use this structure exactly. Each language block is collapsible. EN is open by default; VI and JP are collapsed.

```markdown
<!-- 🇬🇧 ENGLISH (canonical) -->
## 🎯 Why
<link spec + 1–2 sentences on business goal>

## 🛠️ What
- <change 1>
- <change 2>
- <change 3>

## 🚦 Impact
🟡 Medium — affects <modules>

## 🧪 How to test
1. <step 1>
2. <step 2>
3. <expected result>

## 📸 Screenshots / Demo
<if UI>

## 📐 Spec
- BD: [docs/specs/...](../docs/specs/...)
- DDD: [docs/ddd/...](../docs/ddd/...)

## ✅ Checklist
- [x] Spec FINAL
- [x] Tests added (unit / integration / e2e)
- [x] `/review-staged` passed (no 🔴/🟠)
- [x] `/recheck-release` = READY
- [x] Docs updated (README / CHANGELOG / OpenAPI)
- [ ] Feature flag configured (if applicable)
- [ ] Telemetry / metrics added

## ↩️ Rollback plan
<concrete steps if regression: revert PR? toggle flag? db down migration?>

## 👀 Review focus
- <area reviewer should focus on>

---

<details>
<summary>🇻🇳 Tiếng Việt</summary>

## 🎯 Vì sao
<link spec + 1–2 câu mục tiêu kinh doanh>

## 🛠️ Thay đổi gì
- <thay đổi 1>
- <thay đổi 2>

## 🚦 Mức ảnh hưởng
🟡 Medium — ảnh hưởng <modules>

## 🧪 Cách test
1. <bước 1>
2. <bước 2>
3. <kết quả mong đợi>

## 📐 Spec
- BD: docs/specs/...
- DDD: docs/ddd/...

## ✅ Checklist
- [x] Spec FINAL
- [x] Tests đã thêm
- [x] `/review-staged` PASS (không còn 🔴/🟠)
- [x] `/recheck-release` = READY
- [x] Docs đã update
- [ ] Feature flag (nếu cần)
- [ ] Telemetry / metrics

## ↩️ Kế hoạch rollback
<các bước cụ thể>

## 👀 Reviewer chú ý
- <phần cần soi kỹ>

</details>

<details>
<summary>🇯🇵 日本語</summary>

## 🎯 なぜ (Why)
<spec へのリンク + ビジネス目的 1〜2 文>

## 🛠️ 変更内容 (What)
- <変更 1>
- <変更 2>

## 🚦 影響度 (Impact)
🟡 Medium — 影響範囲: <modules>

## 🧪 テスト手順 (How to test)
1. <手順 1>
2. <手順 2>
3. <期待される結果>

## 📐 仕様書 (Spec)
- BD: docs/specs/...
- DDD: docs/ddd/...

## ✅ チェックリスト
- [x] Spec FINAL
- [x] テスト追加 (unit / integration / e2e)
- [x] `/review-staged` 合格
- [x] `/recheck-release` = READY
- [x] ドキュメント更新
- [ ] フィーチャーフラグ設定 (該当時)
- [ ] テレメトリ / メトリクス追加

## ↩️ ロールバック計画 (Rollback plan)
<具体的な手順>

## 👀 レビュー重点 (Review focus)
- <レビュアーに確認してほしい箇所>

</details>
```

## ⚠️ Hard rules
- DO NOT fabricate spec links — verify they exist before linking.
- Title ≤ 72 chars; body wraps at 100 chars.
- For impact 🟠 / 🔴 — Rollback plan MUST have concrete steps, NOT just "revert PR".
- If PR > 500 lines diff → warn "consider splitting" but still output the PR.
- VI and JP blocks must mirror the EN content; do not omit sections.

## 📤 Output format
3 separate blocks with code fences (```), with headers `### 1. Title`, `### 2. Commit msg`, `### 3. Description` so the user can copy each quickly.
