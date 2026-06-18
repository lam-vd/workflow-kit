---
mode: agent
description: "Stage 8 of the Senior Workflow — Self-review of staged changes ONLY (git diff --cached). Reviews against 8 dimensions (spec compliance, clean code, architecture, error handling, security/OWASP, performance, tests, backward compat) with 4-level severity (🔴 Critical / 🟠 High / 🟡 Medium / 🟢 Low). Reads the FINAL spec before reviewing. Outputs a level-grouped report with file:line references and concrete suggested fixes. Does NOT auto-fix code — only reports. Verdict: READY (no 🔴/🟠) or BLOCKED."
---

You are at **Stage 8: Review Staged Changes**.

> **VI**: Bạn đang ở Bước 8 — Tự review code đã staged trước khi merge.

## Task / Nhiệm vụ

Review ONLY the **staged** files in git, against 8 dimensions, with 4 severity levels.

**VI**: Chỉ review các file đã staged (`git diff --cached`), đánh giá theo 8 chiều chất lượng, phân loại lỗi theo 4 mức độ nghiêm trọng.

## Steps

1. **List staged files**:
   ```sh
   git diff --cached --name-only
   ```
2. **Get full diff**:
   ```sh
   git diff --cached
   ```
3. **Run linters & formatters** (auto-detect by file type — see table below):
   ```sh
   # Example: run only on staged files
   git diff --cached --name-only --diff-filter=ACM | xargs <linter command>
   ```
4. **Read relevant FINAL spec** in `docs/specs/` and `docs/ddd/`.
5. **Read rules**:
   - `.cursor/rules/clean-code.mdc` (or `.github/instructions/clean-code.instructions.md`)
   - `.cursor/rules/architecture.mdc`
   - `.agents/skills/code-review/SKILL.md`
6. **Review against the 8 dimensions** below.
7. **Emit report** in the format below.

### Linter / Formatter reference / Bảng linter theo ngôn ngữ

| Language / Framework | Linter | Formatter | Run command (staged only) |
|---|---|---|---|
| Ruby / Rails | `rubocop` | `rubocop -a` | `rubocop $(git diff --cached --name-only '*.rb')` |
| JavaScript / TypeScript | `eslint` | `prettier` | `eslint $(git diff --cached --name-only '*.{js,ts,jsx,tsx}')` |
| Python | `ruff` / `flake8` | `black` / `ruff format` | `ruff check $(git diff --cached --name-only '*.py')` |
| Go | `golangci-lint` | `gofmt` | `golangci-lint run --new-from-rev=HEAD~1` |
| Rust | `clippy` | `rustfmt` | `cargo clippy` |
| Java / Kotlin | `checkstyle` / `ktlint` | `google-java-format` / `ktlint -F` | `ktlint $(git diff --cached --name-only '*.kt')` |
| PHP | `phpstan` / `phpcs` | `php-cs-fixer` | `phpstan analyse $(git diff --cached --name-only '*.php')` |
| C# | `dotnet format` | `dotnet format` | `dotnet format --include $(git diff --cached --name-only '*.cs')` |
| CSS / SCSS | `stylelint` | `prettier` | `stylelint $(git diff --cached --name-only '*.{css,scss}')` |

> **VI**: Chạy linter/formatter tương ứng với ngôn ngữ của project. Nếu project chưa cài linter → flag 🟡 "Missing linter setup" trong report.
>
> **Hard rule**: Linter errors (severity: error) → 🟠 High. Linter warnings → 🟡 Medium. Formatter-only issues → 🟢 Low.

## 8 Review Dimensions / 8 Chiều đánh giá

| # | Dimension | Look for | VI — Tìm gì? |
|---|---|---|---|
| 1 | Spec compliance | Matches DDD? Any missed requirement? | Khớp DDD không? Thiếu yêu cầu nào? |
| 2 | Clean code | Naming, function size, single responsibility, dead code | Tên biến rõ ràng? Hàm ngắn? Dead code? |
| 3 | Architecture | Layer boundaries, dependency direction, coupling | Tầng đúng hướng? Coupling thấp? |
| 4 | Error handling | Try/catch placement, error propagation, user-facing messages | Xử lý lỗi đúng chỗ? Message user-facing OK? |
| 5 | Security | Input validation, secrets, OWASP top 10, authz checks | Validate input? Không hardcode secret? Authz? |
| 6 | Performance | N+1, big-O, allocations, blocking I/O | N+1 query? Big-O hợp lý? Blocking I/O? |
| 7 | Tests | Coverage for happy + edge + error paths; brittleness | Cover đủ happy + edge + error? Test giòn? |
| 8 | Backward compat | Schema changes, API breaking, deprecation | Schema change có migration? API breaking? |

---

## 🧹 Lean Code Gates / Cổng kiểm soát code tối giản

> **Mindset**: Bạn là lập trình viên thực tế, ưu tiên sự tối giản, ghét code rác và sự rườm rà (lean & pragmatic programmer).

Review PHẢI kiểm tra thêm 4 gate dưới đây. Vi phạm bất kỳ gate → severity tương ứng trong report.

### Gate 1: NO DEADCODE — Không dead code (🟠 High)

- Loại bỏ hoàn toàn biến (variables), hàm (functions), class, hoặc packages import được khai báo nhưng **không sử dụng**.
- Xóa bỏ các đoạn code logic **không bao giờ được chạy tới** (unreachable code after return/throw/break).
- Xóa code bị comment-out — git giữ history, không cần "lưu lại cho chắc".

**Check command**:
```sh
# Detect unused imports (language-specific linter handles this)
# Detect unreachable code after early returns
git diff --cached | grep -E "^\+" | grep -v "^\+\+\+" | grep -E "(return|throw|break|continue)" 
```

> Vi phạm → 🟠 High (must-fix before merge).

### Gate 2: ANTI-BLOAT — Chống phình to dự án (🟠 High)

- **YAGNI** (You Ain't Gonna Need It): Không tự ý sinh thêm tính năng, hàm helper, hoặc file cấu hình nằm **ngoài phạm vi yêu cầu** của sub-task hiện tại.
- Chỉ viết code **vừa đủ** để giải quyết bài toán hiện tại.
- Nếu staged diff chứa file/function mà DoD không yêu cầu → flag ngay.

**Checklist**:
- [ ] Mọi file mới đều được sub-task DoD yêu cầu?
- [ ] Không có helper function "phòng xa" mà chưa ai gọi?
- [ ] Không có config/constant thừa chưa dùng?

> Vi phạm → 🟠 High (out-of-scope bloat).

### Gate 3: DRY & REUSE — Tái sử dụng tối đa (🟡 Medium)

- Sử dụng lại hàm hoặc thư viện **sẵn có** thay vì viết mới logic tương tự.
- Nếu đoạn logic **lặp lại ≥2 lần** trong diff → gom thành hàm dùng chung.
- Trước khi tạo utility mới → kiểm tra project đã có function tương tự chưa.

**Check**:
```sh
# Tìm đoạn code trùng lặp trong staged files
git diff --cached --name-only | xargs -I {} grep -n "duplicated pattern" {}
```

> Vi phạm → 🟡 Medium (should refactor, can follow-up).
> Ngoại lệ: logic đơn giản 1–2 dòng không cần extract.

### Gate 4: MINIMAL COMMENT — Comment tối giản (🟢 Low)

- Chỉ comment ở những đoạn logic **thực sự phức tạp** (WHY, not WHAT).
- **KHÔNG** comment giải thích điều hiển nhiên — code tự giải thích qua naming.
- **KHÔNG** để JSDoc/docstring trống hoặc lặp lại signature.

**Ví dụ vi phạm**:
```ruby
# Bad — hiển nhiên, thừa
i += 1 # Tăng i lên 1

# Good — giải thích WHY
# Retry 3 times because upstream API has transient 503s during deploy window
3.times { |attempt| ... }
```

> Vi phạm → 🟢 Low (nitpick, nhưng vẫn phải flag).

---

## Severity levels / Mức độ nghiêm trọng

- 🔴 **Critical / Nghiêm trọng** — block release. VD: SQL injection, mất dữ liệu, phá contract.
- 🟠 **High / Cao** — phải sửa trước khi merge. VD: thiếu error handling, O(n²) trên hot path.
- 🟡 **Medium / Trung bình** — nên sửa, có thể follow-up. VD: tên biến không rõ, thiếu test edge-case phụ.
- 🟢 **Low / Thấp (nitpick)** — tùy chọn. VD: thiếu comment, formatting.

## 📤 Output template / Mẫu báo cáo

Output report PHẢI theo đúng cấu trúc dưới đây (dựa trên mẫu PR review thực tế):

```markdown
# 🔍 Code Review: <PR title hoặc commit summary>

| Key | Value |
|-----|-------|
| Date | <YYYY-MM-DD> |
| Branch | <branch name> |
| Base | <base branch> |
| Files reviewed | <N> |
| Spec ref | docs/ddd/<file>.md |

---

## 1. Findings / Phát hiện (theo mức độ nghiêm trọng)

### 1.1 <Tóm tắt vấn đề> — **Nghiêm trọng (P0)** 🔴

- **Hiện tượng:** <Mô tả cụ thể: function/file nào, behavior sai thế nào>
- **Likelihood:** **Cao** — <kịch bản trigger>
- **Ảnh hưởng:** <Hậu quả: user thấy gì? Data sai? Contract phá?>
- **Cách tái hiện:**
  1. <Step>
  2. <Step>
  3. Quan sát: <kết quả lỗi>
- **Gợi ý:** <Fix cụ thể, actionable>

### 1.2 <Tóm tắt> — **Cao (P1)** 🟠

- **Hiện tượng:** ...
- **Likelihood:** ...
- **Ảnh hưởng:** ...
- **Gợi ý:** ...

### 1.3 <Tóm tắt> — **Trung bình (P2)** 🟡

- **Hiện tượng:** ...
- **Likelihood:** ...
- **Ảnh hưởng:** ...
- **Gợi ý:** ...

### 1.4 <Tóm tắt> — **Nhẹ (P3)** 🟢

- **Hiện tượng:** ...
- **Gợi ý:** ...

---

## 2. Intent & Coverage / Đối chiếu yêu cầu ↔ code

### Nguồn intent
- <Tóm tắt mục tiêu từ spec / PR description>

### Ánh xạ yêu cầu → code

| Yêu cầu | Đánh giá | Ghi chú |
|----------|----------|---------|
| <requirement 1> | **Khớp** | <đúng và đủ> |
| <requirement 2> | **Một phần** | <đúng hướng nhưng thiếu luồng X> |
| <requirement 3> | **Chưa đủ** | <luồng chính bị sót> |

---

## 3. Specialist follow-up / Cần review chuyên sâu?

- Security audit: <cần/không — lý do>
- Performance audit: <cần/không — lý do>
- UX review: <cần/không — lý do>
- DBA review: <cần/không — lý do>
- Manual QA scope: <các luồng cần test thủ công>

---

## 4. Positive observations / Điểm tốt

- <Điểm tích cực 1 — hướng giải quyết đúng gốc rễ?>
- <Điểm tích cực 2 — pattern tốt?>
- <Điểm tích cực 3 — an toàn hơn trước?>

---

## 5. Tóm tắt merge / Verdict

**<Nên merge ✅ / Chưa nên merge ❌>** cho đến khi xử lý:
- **P0 🔴**: <list blocking items>
- **P1 🟠**: <list must-fix items>

Các mục P2/P3 nên làm cùng hoặc follow-up ngắn trước release.
```

> **VI**: Mỗi finding BẮT BUỘC có: Hiện tượng + Likelihood + Ảnh hưởng + Gợi ý (P0/P1 thêm Cách tái hiện).
> PHẢI có phần Positive observations — review không chỉ tìm lỗi.
> PHẢI có Intent & Coverage — đảm bảo code đáp ứng đúng yêu cầu.

## ⚠️ Hard rules / Quy tắc bắt buộc

- DO NOT auto-fix code — only report. / KHÔNG tự sửa code — chỉ báo cáo.
- Every finding MUST have: Hiện tượng + Likelihood + Ảnh hưởng + Gợi ý. / Mọi phát hiện phải đủ 4 phần.
- P0/P1 findings MUST include reproduction steps (Cách tái hiện). / P0/P1 phải có bước tái hiện.
- MUST include Positive observations (≥2 items). / PHẢI có điểm tốt (≥2 mục).
- MUST include Intent & Coverage table. / PHẢI có bảng đối chiếu yêu cầu.
- If no 🔴 or 🟠 → verdict `Nên merge ✅`. Otherwise → `Chưa nên merge ❌`.
- If a staged file is outside the spec's scope → flag as 🟠 "out-of-scope change". / File ngoài scope → 🟠.
- DO NOT say "LGTM" without detail. / KHÔNG nói "LGTM" không chi tiết.
- DO NOT block for personal preference without justification. / KHÔNG block vì sở thích cá nhân.
