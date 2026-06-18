---
name: code-review
description: "Skill for performing self or peer code/PR review at Stage 8 of the Senior Workflow. Follows a structured methodology: (1) Findings grouped by severity with Hiện tượng/Likelihood/Ảnh hưởng/Cách tái hiện/Gợi ý per item; (2) Intent & Coverage mapping (requirements → code); (3) Specialist follow-up; (4) Positive observations; (5) Merge verdict. Critic mindset — review the diff, distinguish must-fix from nice-to-have, propose concrete fixes. 8 dimensions + 4 Lean Code Gates. 4-level severity (🔴 P0 Critical / 🟠 P1 High / 🟡 P2 Medium / 🟢 P3 Low). Flags out-of-scope changes. Anti-patterns to avoid as reviewer."
---

# Skill: Code Review / PR Review

## Mindset / Tư duy

- **Critic hat on** — không phải approval-stamp.
- Review **the diff** (không đọc lại cả repo).
- Phân biệt rõ `must-fix` vs `nice-to-have`.
- **Propose concrete fixes** — không chỉ phàn nàn mà phải gợi ý cụ thể.
- **Lean & pragmatic** — ghét code rác, thừa, rườm rà.
- **Acknowledge good work** — ghi nhận điểm tốt, không chỉ tìm lỗi.

---

## Review Methodology / Phương pháp review

### Phase 1: Context gathering / Thu thập ngữ cảnh

1. Đọc PR description / commit message → hiểu intent.
2. Đọc FINAL spec (BD + DDD) nếu có.
3. List files changed, classify by layer (domain / application / infra / UI).
4. Chạy linters (rubocop, eslint, ruff, etc.) trên staged/changed files.

### Phase 2: Review against dimensions / Đánh giá theo chiều

Review 8 dimensions + 4 Lean Code Gates (xem bên dưới).

### Phase 3: Intent & Coverage / Đối chiếu yêu cầu ↔ code

Map từng yêu cầu trong spec/PR description → code tương ứng:
- **Khớp** — code implement đúng và đủ.
- **Một phần** — implement đúng hướng nhưng thiếu edge case / luồng phụ.
- **Chưa đủ** — luồng chính hoặc phụ bị bỏ sót.
- **Ngoài scope** — code không thuộc yêu cầu nào → flag.

### Phase 4: Specialist follow-up / Cần review chuyên sâu?

Xác định xem PR có cần thêm review từ chuyên gia không:
- Security audit (nếu có auth/payment/PII)?
- Performance audit (nếu có query phức tạp / high-traffic path)?
- UX review (nếu có thay đổi UI)?
- DBA review (nếu có migration)?

### Phase 5: Verdict / Kết luận merge

---

## 8 Review Dimensions / 8 Chiều đánh giá

### 1. Spec compliance / Tuân thủ spec
- Code khớp DDD? Thiếu requirement nào?
- Có thay đổi ngoài scope? → phải tách PR hoặc update spec.

### 2. Clean code / Code sạch
- Tên biến/hàm có ý nghĩa?
- Hàm/file size hợp lý?
- Magic values, dead code, commented-out blocks?
- Import gọn gàng?

### 3. Architecture / Kiến trúc
- Vi phạm layer boundaries? (Domain import HTTP/SQL?)
- Dependency direction đúng?
- Circular imports?
- Mutable global singletons?

### 4. Error handling / Xử lý lỗi
- Entry point có error boundary?
- Error có code/type cụ thể?
- Log có context (correlationId, params), KHÔNG log secrets?
- Exception bị nuốt (swallowed)?

### 5. Security / Bảo mật
- Input validation at boundaries?
- Authz check on new endpoints?
- Secrets / keys not hardcoded?
- SQL injection / XSS / SSRF / IDOR?
- OWASP Top 10:
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

### 6. Performance / Hiệu năng
- N+1 queries?
- Big-O hợp lý cho data size thực tế?
- Allocations trong hot loops?
- Blocking I/O trong async context?

### 7. Tests / Kiểm thử
- Coverage cho happy + edge + error paths?
- Tên test mô tả behavior?
- Test giòn (order-dependent, time-dependent, real network)?
- Mock chain phức tạp → refactor smell?

### 8. Backward compatibility / Tương thích ngược
- Schema change có migration plan?
- API breaking → versioned / deprecated?
- Config change có default cho old instances?

---

## 4 Lean Code Gates / Cổng code tối giản

### Gate 1: NO DEADCODE (🟠 High)
- Unused variables, functions, classes, imports → xóa.
- Unreachable code (after return/throw/break) → xóa.
- Commented-out code → xóa (git giữ history).
- Expired feature flags → xóa.

### Gate 2: ANTI-BLOAT / YAGNI (🟠 High)
- Không sinh thêm features/helpers/configs ngoài scope sub-task.
- Chỉ code **vừa đủ** giải quyết bài toán hiện tại.
- File/function mà DoD không yêu cầu → flag.

### Gate 3: DRY & REUSE (🟡 Medium)
- Dùng lại hàm/thư viện sẵn có thay vì viết mới logic tương tự.
- Logic lặp ≥2 lần → gom thành hàm chung.
- Ngoại lệ: logic 1–2 dòng trivial không cần extract.

### Gate 4: MINIMAL COMMENT (🟢 Low)
- Comment chỉ ở logic phức tạp (WHY, not WHAT).
- Không comment điều hiển nhiên.
- Không JSDoc/docstring trống hoặc lặp signature.

---

## 4-level Severity / 4 mức nghiêm trọng

| Level | Label | When / Khi nào | Ví dụ |
|---|---|---|---|
| 🔴 | P0 Critical | Block release — không merge được | SQL injection, mất dữ liệu, phá contract, luồng chính không chạy |
| 🟠 | P1 High | Must-fix trước merge | Thiếu error handling, O(n²) hot path, thiếu authz, dead code, out-of-scope bloat |
| 🟡 | P2 Medium | Should-fix, có thể follow-up | Tên biến chưa rõ, thiếu test edge-case phụ, DRY violation |
| 🟢 | P3 Low / nitpick | Tùy chọn | Comment thừa, formatting, naming preference |

---

## Finding structure / Cấu trúc mỗi phát hiện

Mỗi finding PHẢI có đủ 5 phần (theo mẫu PR review thực tế):

```markdown
### <Tóm tắt 1 dòng> — **<Severity label> (P<n>)**

- **Hiện tượng**: <Mô tả cụ thể vấn đề gì xảy ra, ở đâu trong code (file:line hoặc function name)>
- **Likelihood / Khả năng xảy ra**: **Cao / Trung bình / Thấp / Có điều kiện** — <giải thích kịch bản trigger>
- **Ảnh hưởng**: <Hậu quả nếu không fix — user thấy gì? Data sai thế nào? UX lệch ra sao?>
- **Cách tái hiện** (nếu applicable):
  1. Step 1
  2. Step 2
  3. Quan sát: <kết quả lỗi>
- **Gợi ý / Fix**: <Đề xuất cụ thể, actionable — code approach hoặc direction>
```

> **VI**: Không bao giờ chỉ nói "có bug" mà không chỉ ra hiện tượng + cách fix. Không bao giờ chỉ nói "nên refactor" mà không giải thích impact.

---

## Intent & Coverage mapping / Đối chiếu yêu cầu

Phải output bảng map giữa yêu cầu (từ spec/PR description) và đánh giá code:

```markdown
## Intent & Coverage

### Nguồn intent
- <Tóm tắt PR description / spec goals>

### Ánh xạ yêu cầu → code

| Yêu cầu | Đánh giá | Ghi chú |
|----------|----------|---------|
| <requirement 1> | **Khớp** / **Một phần** / **Chưa đủ** | <chi tiết> |
| <requirement 2> | ... | ... |
```

---

## Positive observations / Điểm tốt

PHẢI liệt kê ít nhất 2–3 điểm tích cực:
- Hướng giải quyết đúng gốc rễ?
- Pattern tốt được áp dụng?
- Code an toàn hơn trước?
- Naming/structure cải thiện?

> Mục đích: review không chỉ tìm lỗi — phải ghi nhận effort và hướng đi đúng.

---

## Specialist follow-up / Cần review chuyên sâu

```markdown
## Specialist follow-up
- [ ] Security audit: <cần/không cần — lý do>
- [ ] Performance audit: <cần/không cần — lý do>
- [ ] UX review: <cần/không cần — lý do>
- [ ] DBA review: <cần/không cần — lý do>
- Manual QA scope: <liệt kê luồng cần test thủ công>
```

---

## Merge verdict / Kết luận

```markdown
## Tóm tắt merge

**<Nên merge / Chưa nên merge>** cho đến khi xử lý:
- **P0**: <list blocking items>
- **P1**: <list must-fix items>

Các mục P2/P3 nên làm cùng hoặc follow-up ngắn trước release.
```

---

## Reviewer anti-patterns / Điều KHÔNG được làm

- "LGTM" không detail → không phải review.
- Comment theo preference cá nhân không có justification (`I prefer X`).
- Yêu cầu refactor ngoài scope PR → file separate issue.
- Block PR vì style preference, không phải bug.
- Chỉ tìm lỗi mà không ghi nhận điểm tốt.
- Finding không có "Gợi ý" cụ thể → không actionable → không hữu ích.
