# 🧭 Senior Workflow Kit

> **EN**: Personal workflow kit for senior engineers — from task intake to merged PR. Reduces input ambiguity, enforces *think-before-code*, and documents every decision.
>
> **VI**: Bộ workflow cá nhân cho Senior Engineer — từ lúc nhận task đến khi merge PR. Giảm mơ hồ đầu vào, buộc *suy nghĩ trước, code sau*, và tài liệu hoá mọi quyết định.

Works in parallel with **VS Code Copilot** (`.github/prompts`, `.github/instructions`) and **Cursor** (`.cursor/rules`), plus shared **Skills** (`.agents/skills`).

---

## 🚀 Quick Start

1. Open Copilot Chat in VS Code (`chat.promptFiles` enabled).
2. Type `/analyze-task` and paste the task description.
3. Walk through the **9 stages** in [SENIOR-WORKFLOW.md](docs/workflow/SENIOR-WORKFLOW.md).
4. When done → `/create-pr`.

---

## 📁 Structure

```
senior-workflow-kit/
├── AGENTS.md                          # Master orchestrator (read first)
├── README.md
├── common/
│   ├── README.md                      # Shared reusable assets
│   ├── snippets/
│   │   ├── pr-description.trilingual.md
│   │   └── spec-section.trilingual.md
│   └── checklists/
│       └── recheck-spec-scorecard.md
├── .github/
│   ├── copilot-instructions.md
│   ├── prompts/                       # 9 slash commands
│   └── instructions/                  # Auto-applied rules (by glob)
├── .cursor/
│   └── rules/                         # Cursor rules (.mdc)
├── .agents/
│   └── skills/                        # Domain knowledge SKILL.md
└── docs/
    ├── workflow/SENIOR-WORKFLOW.md    # Main workflow doc
    ├── specs/_TEMPLATE.md             # Basic Design template (VI/EN/JP)
    ├── ddd/_TEMPLATE.md               # Detail Design template (VI/EN/JP)
    └── examples/sample-task.md        # End-to-end walk-through
```

---

## 🎯 9 Stages Summary

| # | Stage | Command | Output |
|---|---|---|---|
| 1 | Intake & Analyze | `/analyze-task` | Initial analysis |
| 2 | Grooming (5W + risk) | `/grooming` | Risk matrix + open questions |
| 3 | Write Spec | `/write-spec` | `docs/specs/*.md`, `docs/ddd/*.md` (VI/EN/JP) |
| 4 | Recheck Spec (score) | `/recheck-spec` | Audit scorecard /10 |
| 5 | Final Spec Lock | `/check-spec` | Status = `FINAL` (gate ≥8) |
| 6 | Task Breakdown | `/breakdown-task` | Prioritized sub-tasks |
| 7 | Implement | (code) | Code + tests |
| 8 | Self-review | `/review-staged` | Level-based report |
| 9 | Release + PR | `/recheck-release` → `/create-pr` | PR (VI/EN/JP) |

---

## How To Use (Rules + Skills)

### 1) Daily operational flow

1. Start every task with `/analyze-task` and `/grooming`.
2. Write BD + DDD via `/write-spec` using tri-lingual structure.
3. Recheck quality by `/recheck-spec` (score gate >= 8.0).
4. Lock by `/check-spec`, then implement by sub-task.
5. Before PR: `/review-staged` -> `/recheck-release` -> `/create-pr`.

### 2) How rules are applied

- Copilot rules auto-load from `.github/instructions/` by `applyTo` glob.
- Cursor rules auto-load from `.cursor/rules/` by `globs`.
- In practice:
  - Writing code -> clean-code + architecture rules are active.
  - Writing tests -> testing rules are active.
  - Preparing PR -> pr-conventions are active.

### 3) How skills are applied

- Skills are domain playbooks in `.agents/skills/` and are consulted at the right stages:
  - Stage 3: `writing-bd`, `writing-ddd`
  - Stage 7: `design-patterns`
  - Stage 8: `code-review`
  - Stage 9: `pr-conventions`

### 4) How to use the common folder

- Use `common/snippets/spec-section.trilingual.md` when drafting BD/DDD sections.
- Use `common/checklists/recheck-spec-scorecard.md` when scoring Stage 4.
- Use `common/snippets/pr-description.trilingual.md` when drafting PR body.
- See usage notes in `common/README.md`.

---

## 🚦 Impact Levels

| Level | Color | Meaning / Ý nghĩa |
|---|---|---|
| L1 | 🟢 Low | 1 module, no public API change / 1 module, không đụng API public |
| L2 | 🟡 Medium | 2–4 modules, no schema/contract change / 2–4 modules, không đổi schema/contract |
| L3 | 🟠 High | API / schema / cross-team / API public, schema, hoặc cross-team |
| L4 | 🔴 Critical | Production data, security, payment, migration / Dữ liệu prod, bảo mật, thanh toán, migration |

---

## 📊 Spec Quality Gate (Stage 4)

Audit scoring rubric: **10 items × 1 point**. Pass threshold = **≥ 8.0 / 10**.

| Score | 🔴 Critical | Action |
|---|---|---|
| ≥ 8.0 | 0 | ✅ Auto APPROVED → `/check-spec` |
| ≥ 8.0 | ≥ 1 | ❌ BLOCKED — fix 🔴 first |
| < 8.0 | 0 | ⚠️ Ask user: fix or bypass? Bypass logged in Decision Log |
| < 8.0 | ≥ 1 | ❌ BLOCKED |

Detail: [.github/prompts/recheck-spec.prompt.md](.github/prompts/recheck-spec.prompt.md).

---

## 🌐 Multilingual Outputs

Final deliverables ship in **3 languages** (English canonical, Vietnamese & Japanese mirrors):

- **Spec files** (BD + DDD): each section duplicated for `EN / VI / JP`.
- **PR title**: English (Conventional Commits).
- **PR description & commit body**: 3 collapsible blocks `EN / VI / JP`.

Why? Cross-team / global review readiness without context loss. / Để team đa quốc gia review không mất context.

---

## 🔧 Setup

### VS Code (Copilot)
Add to `settings.json`:
```json
{
  "chat.promptFiles": true,
  "chat.instructionsFilesLocations": { ".github/instructions": true },
  "chat.promptFilesLocations": { ".github/prompts": true }
}
```

### Cursor
`.cursor/rules/*.mdc` auto-loaded by `globs` in YAML frontmatter.

---

## 📜 License
MIT — free to use, PRs welcome.
