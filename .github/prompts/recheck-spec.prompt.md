---
mode: ask
description: "Stage 4 of the Senior Workflow — Critic-mode self-audit of BD + DDD with QUANTITATIVE SCORING (10-item rubric, max 10.0) and severity classification (🔴/🟠/🟡/🟢). Pass threshold = ≥ 8.0/10. If score < 8 with no 🔴 — user is the final decision-maker (option 1: revise, option 2: bypass with Decision Log entry). If any 🔴 Critical — auto-block, no bypass possible. Output: scorecard table + issues table + verdict. Does NOT modify the spec, only reports."
---

You are at **Stage 4: Recheck Spec**.

## Task
Read the BD and DDD files produced in Stage 3. Put on the **critic hat**, find issues, and **score** using the 10-item rubric.

Before scoring, load and reuse the shared checklist at `common/checklists/recheck-spec-scorecard.md` to keep scoring consistent across tasks.

## 🎯 Pass criteria
- **Score ≥ 8.0 / 10** → can proceed to `/check-spec`.
- **Score < 8.0** → spec **NOT YET MEETING BAR**, recommend back to `/write-spec`.
- ⚠️ **User is the final decision-maker**: even with < 8, the user can choose to "bypass" or "fix". You MUST ask explicitly and record the decision.

---

## 📊 Scoring Rubric — 10 items × 1 point

Each item scored as one of three levels:
- **1.0** = Fully passes
- **0.5** = Partial (present but quality below bar)
- **0.0** = Fails / missing

| # | Item | Pass (1.0) | Partial (0.5) | Fail (0.0) |
|---|---|---|---|---|
| 1 | **Open Questions resolved** | All Q from Stages 1–2 answered + reflected | Answered but not fully reflected | Still TBD / unclear |
| 2 | **Edge cases coverage** | ≥5 edge cases with defined behavior | 3–4 edge cases | <3 or behavior missing |
| 3 | **Flow + sequence diagrams** | Flowchart + happy + error sequence (Mermaid) | Only happy path or only flowchart | None |
| 4 | **API contract completeness** | Schema + examples + complete error codes table | Schema present but missing examples or error codes | Schema missing |
| 5 | **Migration & rollback** | Up + down + backfill + concrete rollback steps | Up present, down/rollback weak | Missing or N/A unjustified |
| 6 | **Quantitative test plan** | # tests per layer + coverage target + breakdown | Present but not quantified | Generic "will write tests" |
| 7 | **Observability plan** | Metrics + logs + traces + alert thresholds | Listed generally only | Missing |
| 8 | **Security considerations** | AuthZ + input validation + relevant OWASP + PII handling | 1–2 items | Not addressed |
| 9 | **Performance budget** | p50 / p95 / p99 + concrete throughput | Qualitative only ("fast") | Missing |
| 10 | **Consistency & clarity** | BD ↔ DDD consistent, no "TBD", new dev can read & implement | A few unclear spots | Contradictions / many TBD |

> **Total**: sum of 10 items → max 10.0.
> Some tasks may not need item 5 (no schema change) or item 9 (no perf concern). Items can be marked **N/A with justification**, then final score = (sum of applied items) / (count of applied items) × 10.
> Example: 8 items apply, you score 7.0 → final = 7.0 / 8 × 10 = **8.75**.

---

## 🚦 Severity classification (parallel to score)

| Severity | When | Effect on score |
|---|---|---|
| 🔴 Critical | Spec has core logic error, missing main requirement, security hole | Related item = 0.0 + auto-block |
| 🟠 High | Missing error sequence path, missing rollback, missing authz | Related item = 0.0 |
| 🟡 Medium | Unclear naming, missing minor example | Related item = 0.5 |
| 🟢 Low | Typo, formatting | No deduction |

---

## 📤 Output format

```markdown
# 🔍 Spec Audit Report
**Files**: docs/specs/<file>.md, docs/ddd/<file>.md
**Audit date**: <date>
**Auditor**: <you / agent>

## 📊 Scorecard

| # | Item | Score | Note |
|---|---|---|---|
| 1 | Open Questions resolved | 1.0 | ✅ All 4 Q reflected |
| 2 | Edge cases coverage | 0.5 | 🟡 Only 4 cases, missing concurrent submit |
| 3 | Flow + sequence diagrams | 1.0 | ✅ Flow + happy + error |
| 4 | API contract | 1.0 | ✅ |
| 5 | Migration & rollback | 0.0 | 🟠 Missing outbox rollback script |
| 6 | Quantitative test plan | 1.0 | ✅ |
| 7 | Observability | 0.5 | 🟡 Missing alert threshold for `bounce_rate` |
| 8 | Security | 1.0 | ✅ |
| 9 | Performance budget | 1.0 | ✅ p95 ≤ 200ms |
| 10 | Consistency & clarity | 1.0 | ✅ |
| | **Total** | **8.0 / 10** | |

## ❌ Issues found
| # | File:Section | Issue | Severity | Suggested fix |
|---|---|---|---|---|
| 1 | DDD §9 Rollback | Missing revert step for outbox table on panic | 🟠 | Add `scripts/drain-outbox.ts` + migration down |
| 2 | DDD §10 Observability | `bounce_rate` lacks alert threshold | 🟡 | Add alert: bounce_rate > 5% in 5 min |
| 3 | BD §3 Edge cases | Missing "concurrent submit" case | 🟡 | Add idempotency key in contract |

## ✅ Passed checks
- API contract: schema + examples + error codes complete
- Sequence diagrams: happy + error
- Performance budget quantified
- Security: authz + input validation present

## 🚦 Verdict

|  |  |
|---|---|
| **Score** | **8.0 / 10** |
| **Threshold** | ≥ 8.0 |
| **🔴 Critical issues** | 0 |
| **🟠 High issues** | 1 |
| **Auto verdict** | ✅ APPROVED (meets threshold) |

### ➡️ Next step
- If score ≥ 8 and no 🔴 → run `/check-spec` to FINAL lock.
- If score < 8 → return to `/write-spec` to fix 🔴 / 🟠 issues.

### ⚠️ User decision required (only when score < 8)
> "Current score is **X.X / 10**, below the 8.0 threshold. Please choose:
>
> 1. ✅ **Return to `/write-spec`** to fix issues (recommended).
> 2. ⚠️ **Bypass** and proceed to `/check-spec` — accept risk, will be logged in BD/DDD Decision Log.
>
> Choose **1** or **2**?"
```

---

## ⚠️ Hard rules

1. **DO NOT modify the spec** — only report issues and suggest fixes.
2. **Score MUST be computed by summing the rubric**, not "by feel". Show every item in the scorecard.
3. **N/A items** require a 1-line justification (e.g. "no schema change → item 5 N/A") and trigger pro-rating.
4. **If any 🔴 Critical exists** → regardless of score, verdict is automatically `BLOCKED — fix 🔴 first`. **User CANNOT bypass** this.
5. **If any 🟠 High exists** → related item is automatically 0.0, cannot be 0.5.
6. **When user chooses bypass (only valid for score < 8 AND no 🔴)** → record in BD/DDD Decision Log:
   ```markdown
   | <date> | Bypass audit at score X.X | User approved despite N 🟠 issues. Risks: <list>. Mitigation deferred to <stage/PR>. | <user> |
   ```

---

## 🔁 Re-audit
After user fixes → re-run `/recheck-spec`. The new report MUST reference the previous one:
- Issues fixed → mark ✅ + raise score on related items.
- Issues remaining → keep as-is.
- New issues → flag 🆕.
