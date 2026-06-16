# Copilot Instructions

> **EN**: Auto-loaded by Copilot for every conversation in this repo. Delegates to [AGENTS.md](../AGENTS.md) as the orchestrator.
> **VI**: Tự động đọc cho mọi cuộc hội thoại trong repo. Uỷ quyền cho [AGENTS.md](../AGENTS.md) làm orchestrator.

## Mandatory Rules / Quy tắc bắt buộc

1. **Read [AGENTS.md](../AGENTS.md) first.** It defines the 9-stage workflow + hard gates.
2. **Always state the current stage** in your response (e.g. `Stage 3 — Write Spec`).
3. **Reference rules per stage**:
   - Code → `.github/instructions/clean-code.instructions.md`, `architecture.instructions.md`.
   - Tests → `.github/instructions/testing.instructions.md`.
   - Spec → `.agents/skills/writing-bd/SKILL.md`, `writing-ddd/SKILL.md`.
4. **Slash commands** live in `.github/prompts/`. Suggest the right one for the user's current stage.
5. **No code without FINAL spec.** If user requests "just write the code" before spec is FINAL → push back, suggest `/analyze-task` first.
6. **Trilingual deliverables**: at stages 3 and 9, outputs are EN (canonical) + VI + JP.

## Style

- Reply in English with Vietnamese mirror when helpful; technical terms keep original.
- Use impact emojis: 🟢 🟡 🟠 🔴.
- Conventional Commits for any commit/PR title suggestion.
- Markdown tables for ≥3-column comparisons.
- File references: relative paths, no backticks around them in body text.
