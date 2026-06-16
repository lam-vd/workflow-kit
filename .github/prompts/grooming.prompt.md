---
mode: ask
description: "Stage 2 of the Senior Workflow — Deep grooming after initial analysis. Use AFTER /analyze-task when the task has a clear summary but needs risk discovery, root-cause exploration, and edge-case enumeration BEFORE writing the spec. Produces: 5 Whys (root cause/goal), Risk Matrix (Tech/Business/Ops × likelihood × impact × mitigation), Dependencies (blocked-by/blocks/external), ≥5 edge cases with expected behavior, Non-functional requirements (perf/security/scalability/observability/i18n/a11y/compliance), and Ask-back questions to PO. Mandatory even for 'simple' tasks — never skip 5 Whys."
---

You are at **Stage 2: Grooming** of the workflow.

**Precondition**: `/analyze-task` (Stage 1) output exists. If not — ask the user to run `/analyze-task` first.

## Task

Dig deeper to find hidden problems BEFORE writing the spec. Output MUST contain all 6 sections below.

### 1. 5 Whys
Drill to root cause / root goal. Don't stop at Why 2 or 3.

```
Why 1: <question> → <answer>
Why 2: <deeper question> → <answer>
Why 3: ...
Why 4: ...
Why 5: ...
=> Root: <what really needs solving>
```

### 2. Risk Matrix
| # | Risk | Type (Tech/Biz/Ops) | Likelihood | Impact | Mitigation |
|---|---|---|---|---|---|
| 1 | ... | Tech | High | 🔴 | ... |

Likelihood: High / Medium / Low.
Impact: 🔴 / 🟠 / 🟡 / 🟢.

### 3. Dependencies
- **Blocked by** / Bị block bởi: which team / system / API?
- **Blocks** / Block ai: who is waiting on this?
- **External**: third-party, vendor, cloud quota?

### 4. Edge cases (≥5)
Each case with expected behavior.

| # | Edge case | Expected behavior |
|---|---|---|
| 1 | Empty input | Return 400 with code `EMPTY_INPUT` |

### 5. Non-functional requirements
- **Performance**: concrete SLA (latency p99, throughput)?
- **Security**: PII / auth / authz / encryption?
- **Scalability**: current data volume + 12-month projection?
- **Observability**: which logs / metrics / traces to add?
- **Accessibility / i18n**: required?
- **Compliance**: GDPR / SOC2 / etc.?

### 6. Ask-back questions
Numbered list of unresolved questions. If none → write "All clear, ready for Stage 3".

## ⚠️ Hard rules
- If Risk Matrix has any 🔴 row — mitigation must be concrete, not "TBD".
- If you cannot list ≥5 edge cases — drill more or flag "needs PO input".
- DO NOT skip 5 Whys even for simple tasks.

## ➡️ Next step
After PO answers all Ask-backs → run `/write-spec`.
