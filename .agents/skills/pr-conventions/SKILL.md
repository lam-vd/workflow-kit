---
name: pr-conventions
description: "Skill for crafting PR titles, commit messages, and PR descriptions at Stage 9. Mandates English Conventional Commits format (feat/fix/refactor/perf/test/docs/build/ci/chore) with imperative mood, ≤72 chars, no trailing period. PR Description ships in 3 LANGUAGES (EN canonical + VI + JP) using collapsible <details> blocks, each containing 8 sections: Why with spec link / What bullet list / Impact with color level / How to test concrete steps / Spec links / Checklist (spec FINAL, tests, review-staged, recheck-release, docs, feature flag, telemetry) / Rollback plan (mandatory for 🟠/🔴) / Review focus. PR-size guide (<200 ideal, 200–500 OK, >500 needs justification, >1000 split). Flags anti-patterns: generic titles, one-line descriptions, mixed unrelated changes, missing rollback for high-impact PRs."
---

# Skill: PR Conventions

## PR / Commit Title

Format: `<type>(<scope>): <imperative summary>`

### Types
| type | When | Example |
|---|---|---|
| `feat` | New feature | `feat(auth): add 2FA via TOTP` |
| `fix` | Bug fix | `fix(scheduler): prevent duplicate posts on retry` |
| `refactor` | Refactor without behavior change | `refactor(api): extract validation to middleware` |
| `perf` | Performance improvement | `perf(db): add index on posts.user_id` |
| `test` | Add/modify tests | `test(auth): cover token refresh edge cases` |
| `docs` | Docs only | `docs(readme): update setup steps` |
| `build` | Build / deps | `build(deps): bump axios to 1.7.0` |
| `ci` | CI/CD | `ci: add coverage gate to PR` |
| `chore` | Other chores | `chore: remove unused config` |

### Rules
- **Imperative mood**: "add", "remove", "update" — NOT "added", "adding", "fixed".
- ≤ 72 chars (counting `type(scope):`).
- DO NOT end with a period.
- English only.
- Scope optional but encouraged (module / feature name).

### Breaking change
Add `!` after type or footer `BREAKING CHANGE: <description>`.
Example: `feat(api)!: change /users response shape`.

## Commit body (squash message)

```
<title>

<paragraph: WHY this change matters — business context>

WHAT changed:
- <bullet 1>
- <bullet 2>

Refs: <task-id or spec link>
```

## PR Description — required, tri-lingual

Ship in 3 languages: EN (canonical, expanded by default) + VI + JP (collapsed `<details>`).
Each language block contains the same 8 sections below.

### 1. 🎯 Why
Link spec + 1–2 sentences on business goal.

### 2. 🛠️ What
Bullet list of main changes (≤7 bullets).

### 3. 🚦 Impact
- Level: 🟢/🟡/🟠/🔴
- Affected modules
- Breaking changes (if any)

### 4. 🧪 How to test
Concrete reviewer-runnable steps:
```
1. Pull branch
2. <command>
3. Expect: ...
```

### 5. 📸 Screenshots / Demo
If UI or API → screenshot / curl example / video.

### 6. 📐 Spec
- BD: link
- DDD: link

### 7. ✅ Checklist
```markdown
- [x] Spec FINAL
- [x] Tests added (unit / integration / e2e)
- [x] /review-staged passed (no 🔴/🟠)
- [x] /recheck-release = READY
- [x] Docs updated
- [ ] Feature flag configured (if applicable)
- [ ] Telemetry / metrics added
```

### 8. ↩️ Rollback plan
- For 🟢/🟡 impact: "Revert PR" may be enough.
- For 🟠/🔴 impact: MUST have concrete steps (toggle flag, revert migration, etc.).

### 9. 👀 Review focus (optional but encouraged)
Guide the reviewer to risky areas.

## PR size guidelines
| Size | Diff lines | Action |
|---|---|---|
| Small | < 200 | Ideal |
| Medium | 200–500 | OK |
| Large | 500–1000 | Needs strong justification |
| Huge | > 1000 | Warning — split |

## Anti-patterns
- Generic title: "Update code", "Fix bug", "WIP".
- PR description with only "see Jira".
- Mixing unrelated changes in one PR.
- Missing rollback plan for 🟠/🔴 impact.
- Force push after review (unless announced first).
