---
mode: agent
description: "Stage 5 of the Senior Workflow — FINAL spec lock. Gates entry to coding (Stage 7). Verifies 5 conditions: scorecard exists, score ≥ 8.0 OR explicit user bypass, NO 🔴 Critical (never bypassable), stakeholder sign-off, spec committed. On pass: flips Status DRAFT → FINAL, adds Locked-at timestamp + Audit-score, appends changelog. If bypass: changelog must list open risks + follow-up plan. Refuses to lock when 🔴 exists regardless of user request."
---

You are at **Stage 5: Final Spec Lock**.

## Preconditions (verify ALL 5)
1. ✅ `/recheck-spec` ran with a clear scorecard.
2. ✅ **Score ≥ 8.0 / 10** OR user has explicitly approved a bypass.
3. ✅ **NO 🔴 Critical issue** (Critical always blocks, regardless of user decision).
4. ✅ Stakeholder / PO sign-off (manual user confirmation).
5. ✅ Spec is committed to the current branch.

If ANY condition fails → STOP, print the reason, ask user to fix.

### 🚦 Score gate logic

| Score | 🔴 Critical | Action |
|---|---|---|
| ≥ 8.0 | 0 | ✅ Proceed to lock FINAL |
| ≥ 8.0 | ≥1 | ❌ BLOCKED — fix 🔴 first, no bypass |
| < 8.0 | 0 | ⚠️ Ask user: "Score X.X < 8. Bypass or revise?" — proceed only if user chooses bypass |
| < 8.0 | ≥1 | ❌ BLOCKED |

## Task

1. Open the BD file `docs/specs/*.md` and DDD file `docs/ddd/*.md` for the task.
2. Change `Status: DRAFT` → `Status: FINAL`.
3. Add line `Locked at: <ISO timestamp>` immediately after `Status`.
4. Add line `Audit score: X.X / 10` immediately after `Locked at`.
5. Append the following changelog block at the bottom of EACH file:

```markdown
---
## Changelog
- <YYYY-MM-DD> — DRAFT → FINAL by <author>
  - Audit score: X.X / 10 via /recheck-spec
  - Stakeholders signed off: <names>
  - [If bypassed] ⚠️ Bypassed at score X.X with user approval. Open risks: <list>. Follow-up: <task/PR>.
```

6. Ask the user: "Have you verified all 5 preconditions? (score ≥ 8 or intentional bypass, no 🔴, sign-off received)" — wait for `yes` before committing the changes.
7. After commit, print:

```
✅ Spec FINAL locked.
- BD: docs/specs/...
- DDD: docs/ddd/...
- Audit score: X.X / 10

➡️ Next: /breakdown-task
```

## ⚠️ Hard rules
- NEVER auto-flip to `FINAL` without user confirmation.
- NEVER lock FINAL while 🔴 Critical exists (even if user requests).
- When user bypasses below 8 — Decision Log + Changelog MUST list open risks + follow-up plan.
- DO NOT modify any other content beyond the 4 changes above (status, locked-at, score, changelog).
- After FINAL — any spec change must create a new file (`docs/specs/<date>-<slug>-v2.md`) or bump version.
