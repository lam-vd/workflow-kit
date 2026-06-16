---
name: code-review
description: "Skill for performing self or peer code review at Stage 8 of the Senior Workflow. Critic mindset — review the diff (not whole repo), distinguish must-fix from nice-to-have, propose concrete fixes (not just complaints). Reviews against 8 dimensions: spec compliance, clean code, architecture, error handling, security (with full OWASP Top 10 checklist), performance, tests, backward compatibility. Uses 4-level severity (🔴 Critical block / 🟠 High must-fix / 🟡 Medium should-fix / 🟢 Low nitpick). Output groups issues by severity with file:line + concrete fix proposal. Flags out-of-scope changes as 🟠. Anti-patterns to avoid as reviewer: bare 'LGTM', preference comments without justification, refactor demands outside PR scope, blocking on personal style."
---

# Skill: Code Review

## Mindset
- **Critic hat on**, not approval-stamp.
- Review **the diff**, no need to re-read the whole repo.
- Distinguish `must-fix` vs `nice-to-have` clearly.
- Propose concrete fixes, not just complaints.

## 8 Dimensions

### 1. Spec compliance
- Does the code match the DDD? Any missed requirement?
- Any out-of-scope changes? → must split PR or update spec.

### 2. Clean code
- Names meaningful?
- Function/file size reasonable?
- Magic values, dead code, commented-out blocks?
- Imports tidy?

### 3. Architecture
- Layer boundaries violated? (Domain importing HTTP/SQL?)
- Dependency direction correct?
- Circular imports?
- Mutable global singletons?

### 4. Error handling
- Each entry point has an error boundary?
- Errors have concrete codes/types?
- Logs include context (correlationId, params), DO NOT log secrets?
- Any swallowed exceptions?

### 5. Security
- Input validation at boundaries?
- Authz check on new endpoints?
- Secrets / keys not hardcoded?
- SQL injection / XSS / SSRF / IDOR?
- OWASP Top 10 checklist:
  - [ ] Broken Access Control
  - [ ] Cryptographic Failures
  - [ ] Injection
  - [ ] Insecure Design
  - [ ] Security Misconfiguration
  - [ ] Vulnerable Components
  - [ ] ID & Auth Failures
  - [ ] Software & Data Integrity
  - [ ] Logging & Monitoring Failures
  - [ ] SSRF

### 6. Performance
- N+1 queries?
- Big-O reasonable for expected data size?
- Allocations in hot loops?
- Blocking I/O in async contexts?

### 7. Tests
- Coverage for happy + edge + error paths?
- Test names describe behavior?
- Brittle tests (order-dependent, time-dependent, real network)?
- Complex mock chains → refactor smell?

### 8. Backward compatibility
- Schema change has migration plan?
- API breaking → versioned / deprecated?
- Config change has default for old instances?

## 4-level Severity

| Level | When | Example |
|---|---|---|
| 🔴 Critical | Block release | SQL injection, data loss, broken contract |
| 🟠 High | Must-fix before merge | Missing error handling, O(n²) hot path, missing authz |
| 🟡 Medium | Should-fix, can follow up | Unclear naming, missing minor edge-case test |
| 🟢 Low / nitpick | Optional | Missing comment, formatting |

## Output template

```markdown
## 🔴 Critical (block release)
- [ ] `path/file.ts:42` — <issue>
  - **Fix**: <concrete suggestion>

## 🟠 High
...

## 🟡 Medium
...

## 🟢 Low / nitpick
...

## ✅ Clean
- file/x.ts
- file/y.ts

## 🚦 Verdict
- [x] BLOCKED — fix 🔴 and 🟠 first
- [ ] READY for next stage
```

## Reviewer anti-patterns
- "LGTM" with no detail → not a review.
- Personal-preference comments without rationale (`I prefer X`).
- Refactor demands outside PR scope → file a separate issue.
- Blocking PR for preference, not a bug.
