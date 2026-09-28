# Review learnings — Realtime AI assist (No.148)

> **Source:** Peer review of `feat/148-realtime-deal-ai` — Stage 8 / branch-peer comments, author replies (ops rake), stub/pre-deploy framing.  
> **VI:** Kinh nghiệm review upgrade/refactor + async surface. Đã nâng skill shared `feature-cutover-orphan-review` (không gắn keyword feature vào kit text) + checklist ahm.

---

## Executive summary / Tóm tắt

| Theme | Reviewer signal | Rule/skill upgraded |
|-------|-----------------|---------------------|
| CRUD UI gone, domain kept | Enqueue/policy “no app callers” | `feature-cutover-orphan-review` — **Ops-only** vs Dead-sure |
| Author adds `rake …:reembed` | Resolve orphan comment after verify | Cutover skill + peer protocol |
| Channel mutate ≠ HTTP policy | IDOR risk on dismiss | CONTRACT-AUTHZ-01; checklist Cable row |
| Polling stops on `completed` | Empty AI panel if reorder only | CONTRACT-ASYNC-01 |
| Schema enum ⊃ prompt output | Dead allow-list labels | CONTRACT-SCHEMA-01 |
| Dual stub flags | Live path needs both off | CONTRACT-ENV-01 |
| FE URL template never read | Unwired POST + dead Stimulus value | Cutover Soft-dead / should |
| Request body UI / `I18n.t` | HARD BAN | existing rspec + checklist |
| Stub / not deployed | Don’t 🔴 missing prod backfill | Cutover hard rule #7 |
| Unused locale / constants / targets | Soft-dead | nit only |
| JSON passthrough getters / “xóa method?” | Keep view API; optional store accessor | `lean-facade-review` |
| `I18n.t` + `default:` JA when keys exist | Drop `default:` — not “add i18n” | `lean-facade-review` + language-boundaries |
| Thin `source_foo?` + helper map | Consolidate UI in helper; Soft-dead unused `?` | `lean-facade-review` |

---

## 1. Cutover classification (transferable)

**Wrong instinct:** “Zero `app/` callers → delete everything.”

**Right instinct:**

| Class | Example from No.148 | Action |
|-------|---------------------|--------|
| Shared-keep | FAQ model + tenant association still feed knowledge search | Keep |
| Ops-only | Enqueue helper + `GenerateEmbeddingsJob` via `rake faqs:reembed` | Keep; document |
| Soft-dead | Unused locale keys, Stimulus targets never declared/read | nit / follow-up |
| Dead-sure (until wired) | HTTP `#create` + index URL template JS never uses | should: wire or delete |

**Verify author reply:** grep rake + `Enqueue*` before resolving a “no callers” thread.

---

## 2. Contract sweep (transferable)

1. **AUTHZ** — Every mutating path (HTTP, Cable/WS, job with user-facing side effect) must use the same policy/scope story.
2. **ASYNC** — Client “done” signals must include every phase the UI still needs; naive job reorder breaks polling.
3. **SCHEMA** — Allow-lists must match producers (tighten enum or expand prompt — don’t leave fiction).
4. **ENV** — Document multi-flag live paths; one stub flag is not enough if transport has its own stub.
5. **SEED** — Local sync embed in fixtures ≠ prod enqueue; intentional split is OK if Decision Log / PR says so.

---

## 3. Reviewer discipline

- Never invent bugs; comment only with grep or runnable repro.
- Pre-deploy / stub DoD → missing prod backfill is Decision Log / 🟡, not merge 🔴.
- After “CRUD moved to another PR”, re-run cutover matrix — do not assume orphans were cleaned.

---

## 4. Prevention checklist (implement + review)

- [ ] Cutover table filled (Dead-sure / Ops-only / Soft-dead / Shared-keep)
- [ ] Ops refresh path exists before deleting web writers
- [ ] Channel/WS mutate authz ≡ HTTP
- [ ] Client stop condition vs server phases mapped
- [ ] Schema/enum vs generator output
- [ ] Dual env flags documented for live provider
- [ ] No UI/`I18n.t` in `spec/requests/**`
- [ ] FE `data-*-url-value` has a JS reader or is removed with the controller
- [ ] Grep locales before “missing i18n”; no redundant `I18n.t` `default:` hardcodes when keys exist
- [ ] View-facing JSON getters kept (or store accessor); no raw hash forced into ERB
- [ ] UI source/kind maps live in one helper — unused `foo?` predicates Soft-dead

## 5. Lean facade (model review follow-up)

**Wrong instinct:** “Passthrough getters / `foo?` are redundant → delete and use hash in views” or “`default:` string means missing i18n.”

**Right instinct:**

| Signal | Action |
|--------|--------|
| Getter used in ERB | Keep (or `store_accessor`); don’t push `content['k']` into templates |
| Getter + I18n/`Array()` | Keep — not thin |
| `I18n.t(key)` + key in JA/EN | Already i18n’d; nit only if hard-coded `default:` remains |
| `kind_foo?` + helper `MAP[kind]` | One UI map; Soft-dead unused predicates |

Canonical: `lean-facade-review`.
