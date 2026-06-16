# Sample Task: Add Email Notification on Order Confirmed

> End-to-end walk-through of all 9 stages on a hypothetical task — to show what each stage's output looks like.

---

## Stage 1 — `/analyze-task`

```markdown
# 📋 Stage 1 — Task Analysis

## Task: Send confirmation email when order status = CONFIRMED

### 🎯 Why
**EN**: Customers currently must refresh the page to know their order is confirmed.
"Where is my order?" tickets account for 18% of weekly support volume.
Goal: cut these tickets by 50% within 4 weeks.

**VI**: Hiện tại customer phải tự refresh trang để biết order được confirm.
Support ticket dạng "where is my order?" chiếm 18% volume tuần qua.
Mục tiêu: giảm 50% ticket loại này trong 4 tuần.

### 📦 Scope
**IN:**
- Email sent the moment order transitions `PENDING → CONFIRMED`.
- Templates in EN + VI driven by `user.locale`.
- Retry up to 3 times if mail provider fails.

**OUT:**
- SMS / push notifications (phase 2).
- Emails for other status changes (SHIPPED, DELIVERED) — separate task.
- Marketing opt-out preference UI.

### 🛣️ Approach Options
| Option | Description | Pros | Cons |
|---|---|---|---|
| A. Inline in order service | call mailer right after commit | Simple | Coupling, slow response, mail loss on crash |
| B. Outbox pattern + worker | write event to outbox table, worker reads & sends | Reliable, decoupled | More infra |
| C. Direct queue publish | publish event to SQS/RabbitMQ | Decoupled | At-most-once if publish fails after commit |

→ **Recommend B** — impact 🟠 (revenue-tied) requires reliability.

### 🚦 Impact: 🟠 High
**Affected**: `order-service`, new `notification-service`, DB schema (outbox table).
**Reasoning**: tied to customer trust + revenue funnel.

### ⏱️ Estimate (Suemori)
- Optimistic: 2d
- Realistic: 4d
- Pessimistic: 7d

### ❓ Open Questions
1. Which mail provider — SendGrid or SES?
2. Should we BCC ops for debug?
3. SLA — how long max from confirm → email arrival?
4. Track open / click rates this round?

### ➡️ Next step
`/grooming` after PO answers.
```

---

## Stage 2 — `/grooming`

```markdown
### 5 Whys
- Why 1: Why send a confirm email? → so the customer knows the order was processed.
- Why 2: Why must they know immediately? → reduces anxiety, reduces support tickets.
- Why 3: Why don't they refresh? → mobile users close the app after checkout.
- Why 4: Why do mobile users close so soon? → old flow has no clear visual confirmation.
- Why 5: Why prioritize customer trust? → impacts Q3 retention OKR.
=> **Root**: Need an async confirmation channel outside the page to reduce anxiety and support cost.

### Risk Matrix
| # | Risk | Type | Likelihood | Impact | Mitigation |
|---|---|---|---|---|---|
| 1 | Bounce / spam folder | Tech | High | 🟠 | verified sender, SPF/DKIM, monitor bounce rate |
| 2 | Worker crash → mail not sent | Ops | Medium | 🟠 | monitoring + alert, idempotent handler |
| 3 | Wrong i18n template → VI customer gets EN mail | Biz | Low | 🟡 | unit test per locale |
| 4 | Provider rate limit → backlog | Tech | Low | 🟡 | exponential backoff |

### Dependencies
- **Blocked by**: Infra team for SES setup (1d).
- **Blocks**: Marketing wants to reuse the template engine — must stabilize first.

### Edge cases
1. Order confirmed → cancelled within 1 minute: still send confirm email? → **Yes** (user triggered confirm); cancellation email is separate.
2. User has unsubscribed (`email_preferences`): **do not send**.
3. Invalid email: **log + alert ops**, no infinite retry.
4. Provider down >5 min: **circuit-break**, accumulate in outbox.
5. Duplicate confirm event (retry): **idempotency** by `order_id + event_type`.

### NFR
- Latency: ≤ 30s from confirm → inbox (p95).
- Reliability: ≥ 99.9% delivery for valid emails.
- Security: email content NEVER includes full credit card; last 4 only.
- Observability: dashboard with send rate, bounce rate, latency.

### Ask-back
✅ All resolved (PO confirmed: SES + 30s SLA + no open/click tracking this round).
```

---

## Stage 3 — `/write-spec`

→ Generate `docs/specs/2026-06-16-order-confirmation-email.md` and `docs/ddd/2026-06-16-order-confirmation-email.md` from the templates.

Both files are **tri-lingual** (EN canonical + VI + JP collapsible blocks).

**BD must include**: 1 high-level flowchart at component/actor level.
**DDD must include**: 1 logical flowchart + 1 happy-path sequence + 1 error-path sequence + 1 ERD (outbox table is new).

(Detailed file contents skipped here — copy from `_TEMPLATE.md` and fill in.)

---

## Stage 4 — `/recheck-spec`

```markdown
# 🔍 Spec Audit Report
**Files**: docs/specs/2026-06-16-order-confirmation-email.md, docs/ddd/2026-06-16-order-confirmation-email.md

## 📊 Scorecard
| # | Item | Score | Note |
|---|---|---|---|
| 1 | Open Questions resolved | 1.0 | ✅ 4/4 reflected |
| 2 | Edge cases coverage | 1.0 | ✅ 5 cases |
| 3 | Flow + sequence diagrams | 1.0 | ✅ flowchart + happy + error |
| 4 | API contract | 1.0 | ✅ |
| 5 | Migration & rollback | 0.0 | 🟠 Missing outbox rollback script |
| 6 | Quantitative test plan | 1.0 | ✅ |
| 7 | Observability | 0.5 | 🟡 Missing alert threshold for `bounce_rate` |
| 8 | Security | 1.0 | ✅ |
| 9 | Performance budget | 1.0 | ✅ p95 ≤ 200ms |
| 10 | Consistency & clarity | 1.0 | ✅ |
| | **Total** | **8.5 / 10** | |

## ❌ Issues
| # | File:Section | Issue | Severity | Fix |
|---|---|---|---|---|
| 1 | DDD §9 | Rollback plan missing outbox revert step | 🟠 | Add migration down + drain script |
| 2 | DDD §10 | `bounce_rate` lacks alert threshold | 🟡 | bounce > 5% in 5 min → alert |

## 🚦 Verdict
| | |
|---|---|
| Score | **8.5 / 10** |
| Threshold | ≥ 8.0 |
| 🔴 Critical | 0 |
| Auto verdict | ✅ APPROVED |
```

→ 8.5 ≥ 8 with no 🔴 → proceed to `/check-spec`. (Still recommended to fix 🟠 if time permits, or track as follow-up.)

> **Alternative case**: if score = 7.0 with 0 🔴 → `/recheck-spec` will ask:
> "Score 7.0 < 8. Choose (1) return to `/write-spec` to fix, or (2) bypass with risks logged?"
> If user picks 2 → log in BD/DDD Decision Log and continue.

---

## Stage 5 — `/check-spec`
→ Flip BD and DDD `Status: DRAFT → FINAL`, write `Locked at`, `Audit score: 8.5 / 10`, append changelog.

---

## Stage 6 — `/breakdown-task`

```markdown
| # | Title | Priority | Est | Risk | Depends | DoD |
|---|---|---|---|---|---|---|
| 1 | Migration: outbox table + index | P0 | 2h | 🟠 | — | up/down OK, rollback verified |
| 2 | Outbox repository + tests | P1 | 3h | 🟢 | 1 | unit ≥85%, idempotency test pass |
| 3 | Order service: emit OrderConfirmed event to outbox | P1 | 3h | 🟡 | 2 | integration test: commit + outbox row in same tx |
| 4 | Notification worker (poll outbox + send via SES) | P1 | 4h | 🟠 | 2 | retry 3x, dead letter, integration test |
| 5 | i18n templates (en, vi) | P2 | 2h | 🟢 | — | snapshot test per locale |
| 6 | Email preferences check (unsubscribe) | P2 | 2h | 🟡 | 4 | unit test happy + opt-out paths |
| 7 | Telemetry: send rate, bounce, latency | P4 | 2h | 🟢 | 4 | metrics visible in staging |
| 8 | Feature flag wiring + rollout doc | P0 | 1h | 🟢 | — | flag toggle end-to-end |
```

→ Order: 1 → 2 → 3 → 4 → 5 → 6 → 7 → 8.

---

## Stage 7 — Implement
- Re-read FINAL spec.
- Read `.cursor/rules/clean-code.mdc` + `architecture.mdc`.
- Code each sub-task with small commits.
- After each sub-task → `/review-staged`.

---

## Stage 8 — `/review-staged` (sample output)

```markdown
## 🟠 High
- [ ] `notification-worker/handler.ts:88` — does not catch error from SES; uncaught promise rejection can crash the worker.
  - **Fix**: try/catch wrap, push to DLQ after 3 retries.

## 🟡 Medium
- [ ] `outbox/repo.ts:42` — query lacks LIMIT, full table scan when backlog is large.
  - **Fix**: add LIMIT 100 + ORDER BY created_at.

## ✅ Clean
- migrations/20260616_outbox.sql
- templates/order-confirmation.en.hbs
- templates/order-confirmation.vi.hbs

## 🚦 Verdict: BLOCKED — fix 🟠 first
```

→ Fix → `/review-staged` again → READY.

---

## Stage 9 — `/recheck-release` + `/create-pr`

**`/recheck-release`** → READY ✅.

**`/create-pr`** generates 3 copy-ready blocks:

### 1. Title
```
feat(notification): send order confirmation email via outbox
```

### 2. Squash commit
```
feat(notification): send order confirmation email via outbox

Reduce "where is my order" support tickets by sending async email
on order confirmation. Uses transactional outbox pattern to ensure
exactly-once delivery.

WHAT changed:
- Add outbox table + repository
- Order service emits OrderConfirmed event to outbox
- New notification worker polls outbox and sends via SES
- i18n templates (en, vi)
- Honors email_preferences.unsubscribed

Refs: TASK-1234
```

### 3. Description (tri-lingual)

```markdown
<!-- 🇬🇧 ENGLISH (canonical) -->
## 🎯 Why
[Spec](../docs/specs/2026-06-16-order-confirmation-email.md) — reduce 50% of "where is my order" tickets within 4 weeks.

## 🛠️ What
- Outbox pattern for reliable email delivery
- Notification worker with 3x retry + DLQ
- i18n templates (en, vi)
- Honors unsubscribe preference

## 🚦 Impact
🟠 High — affects `order-service`, new `notification-service`, schema.

## 🧪 How to test
1. `pnpm dev` — start order + notification services.
2. POST `/orders/123/confirm` with valid auth.
3. Expect: email arrives in test inbox within 30s.
4. Set `email_preferences.unsubscribed = true` → no email sent.

## 📐 Spec
- BD: docs/specs/2026-06-16-order-confirmation-email.md
- DDD: docs/ddd/2026-06-16-order-confirmation-email.md

## ✅ Checklist
- [x] Spec FINAL (audit score 8.5)
- [x] Tests added (unit + integration)
- [x] /review-staged passed
- [x] /recheck-release READY
- [x] Docs updated (README §Notifications)
- [x] Feature flag `feature.order_confirmation_email.enabled`
- [x] Telemetry: send_rate, bounce_rate, latency

## ↩️ Rollback plan
1. Toggle `feature.order_confirmation_email.enabled = false` (instant).
2. Stop notification-worker (no new sends).
3. If schema issue: run migration `down` (clears outbox).
4. Pending events: drain via `scripts/drain-outbox.ts` or discard.

## 👀 Review focus
- `notification-worker/handler.ts` retry/DLQ logic.
- Tx boundary in `order-service/confirm.ts` (outbox + order update same tx).

---

<details>
<summary>🇻🇳 Tiếng Việt</summary>

## 🎯 Vì sao
[Spec](../docs/specs/2026-06-16-order-confirmation-email.md) — giảm 50% support ticket "where is my order" trong 4 tuần.

## 🛠️ Thay đổi gì
- Outbox pattern để gửi email đáng tin cậy
- Notification worker với retry 3 lần + DLQ
- Template i18n (en, vi)
- Tôn trọng preference unsubscribe

## 🚦 Mức ảnh hưởng
🟠 High — ảnh hưởng `order-service`, `notification-service` mới, schema.

## 🧪 Cách test
1. `pnpm dev` — khởi động order + notification service.
2. POST `/orders/123/confirm` với auth hợp lệ.
3. Mong đợi: email vào test inbox trong 30s.
4. Đặt `email_preferences.unsubscribed = true` → không gửi email.

## 📐 Spec
- BD: docs/specs/2026-06-16-order-confirmation-email.md
- DDD: docs/ddd/2026-06-16-order-confirmation-email.md

## ✅ Checklist
- [x] Spec FINAL (audit score 8.5)
- [x] Tests đã thêm (unit + integration)
- [x] /review-staged PASS
- [x] /recheck-release READY
- [x] Docs đã update
- [x] Feature flag `feature.order_confirmation_email.enabled`
- [x] Telemetry: send_rate, bounce_rate, latency

## ↩️ Kế hoạch rollback
1. Toggle `feature.order_confirmation_email.enabled = false` (tức thì).
2. Dừng notification-worker (không gửi mới).
3. Nếu lỗi schema: chạy migration `down` (xoá outbox).
4. Event đang pending: drain qua `scripts/drain-outbox.ts` hoặc discard.

## 👀 Reviewer chú ý
- Logic retry/DLQ ở `notification-worker/handler.ts`.
- Tx boundary ở `order-service/confirm.ts` (outbox + order update cùng 1 tx).

</details>

<details>
<summary>🇯🇵 日本語</summary>

## 🎯 なぜ (Why)
[Spec](../docs/specs/2026-06-16-order-confirmation-email.md) — 4 週間で「注文はどこですか」サポートチケットを 50% 削減。

## 🛠️ 変更内容 (What)
- 信頼性のあるメール配信のための Outbox パターン
- 3 回リトライ + DLQ 付き Notification worker
- i18n テンプレート (en, vi)
- 配信停止 (unsubscribe) 設定の尊重

## 🚦 影響度 (Impact)
🟠 High — 影響範囲: `order-service`、新規 `notification-service`、スキーマ。

## 🧪 テスト手順 (How to test)
1. `pnpm dev` で order + notification サービスを起動。
2. 有効な認証で `POST /orders/123/confirm`。
3. 期待値: 30 秒以内にテスト受信箱にメール到着。
4. `email_preferences.unsubscribed = true` を設定 → メール送信されない。

## 📐 仕様書 (Spec)
- BD: docs/specs/2026-06-16-order-confirmation-email.md
- DDD: docs/ddd/2026-06-16-order-confirmation-email.md

## ✅ チェックリスト
- [x] Spec FINAL (監査スコア 8.5)
- [x] テスト追加 (unit + integration)
- [x] /review-staged 合格
- [x] /recheck-release READY
- [x] ドキュメント更新
- [x] フィーチャーフラグ `feature.order_confirmation_email.enabled`
- [x] テレメトリ: send_rate, bounce_rate, latency

## ↩️ ロールバック計画 (Rollback plan)
1. `feature.order_confirmation_email.enabled = false` を切り替え (即時)。
2. notification-worker を停止 (新規送信なし)。
3. スキーマ起因の場合: migration `down` 実行 (outbox クリア)。
4. 保留中のイベント: `scripts/drain-outbox.ts` でドレインまたは破棄。

## 👀 レビュー重点 (Review focus)
- `notification-worker/handler.ts` のリトライ / DLQ ロジック。
- `order-service/confirm.ts` のトランザクション境界 (outbox と注文更新が同一 tx)。

</details>
```

---

## Summary

This walk-through shows each stage producing a **concrete output** — analysis → spec → code → tri-lingual PR. No stage is "ceremonial"; each one reduces risk for the next.
