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

## 📤 Output template

```markdown
# 🔍 Review Report — <task name / commit>
**Files reviewed**: 7
**Spec ref**: docs/ddd/<file>.md
**Reviewed at**: <timestamp>

## 🔴 Critical (block release)
- [ ] `path/file.ts:42` — SQL query uses string concatenation, exposed to injection.
  - **Fix**: switch to parameterized query.

## 🟠 High (must-fix before merge)
- [ ] `path/file.ts:88` — Does not catch error from external API; user gets 500.
  - **Fix**: wrap try/catch, return 502 with code `UPSTREAM_FAILED`.

## 🟡 Medium (should-fix, can follow up)
- [ ] `path/file.ts:120` — `doStuff` is 80 lines, violates SRP.

## 🟢 Low / nitpick
- [ ] `path/file.ts:5` — Missing JSDoc for public function.

## ✅ Clean (no issues)
- src/utils/format.ts
- src/types/user.ts

## 🚦 Verdict
- [ ] READY for `/recheck-release`
- [x] BLOCKED — fix 🔴 and 🟠 first
```

## ⚠️ Hard rules
- DO NOT auto-fix code — only report.
- Every issue MUST have: `file:line`, brief description, concrete suggested fix.
- If no 🔴 or 🟠 → verdict `READY`. Otherwise → `BLOCKED`.
- If a staged file is outside the spec's scope → flag as 🟠 "out-of-scope change".
