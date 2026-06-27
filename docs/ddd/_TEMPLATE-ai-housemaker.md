# DDD: <Feature name> — ai-housemaker

- **Status**: DRAFT
- **Task**: No.XX
- **BD**: [docs/specs/<YYYY-MM-DD>-<slug>.md](../specs/<YYYY-MM-DD>-<slug>.md)
- **VI (Notion)**: [docs/ddd/<YYYY-MM-DD>-<slug>.vi.md](./<YYYY-MM-DD>-<slug>.vi.md)
- **Impact**: 🟢 / 🟡 / 🟠 / 🔴
- **Created**: YYYY-MM-DD
- **Figma**: file key + node id (if applicable)

> Implementation source of truth. Code drift from this DDD = bug or DDD update required.

---

## 1. Scope & phase goal

**Phase goal:** [One sentence — what users/system achieve after ship]

**In scope:**
- ...

**Out of scope:**
- ...

---

## 2. Current state

| Component | Path | Current behavior |
|---|---|---|
| Route | `config/routes.rb` | ... |
| Controller | `app/controllers/...` | ... |
| View | `app/views/...` | ... |
| Service | `app/services/...` | ... |

**Reuse patterns:**

| Feature | Reference | Notes |
|---|---|---|
| Modal | `app/views/customers/_form_modal.html.erb` | Preline + Turbo Frame |
| ... | ... | ... |

---

## 3. Gap analysis

- [ ] ...
- [ ] ...

---

## 4. Data model

**Migration:** Yes / No

| UI label | DB column | Form attribute | Validation |
|---|---|---|---|
| ... | ... | ... | ... |

---

## 5. Domain & services

### 5.1 Services

| Service | When called | Notes |
|---|---|---|
| `*Manager::*` | ... | ... |

### 5.2 Domain decisions

| Decision | Rationale | Won't do |
|---|---|---|
| ... | ... | ... |

---

## 6. Routes, controllers & forms

### 6.1 Routes

```ruby
# route definitions
```

| HTTP | Path | Action | Notes |
|---|---|---|---|
| ... | ... | ... | ... |

### 6.3 Controllers

[Pseudocode / strong params for non-trivial branches]

---

## 7. Views & UI patterns

```
app/views/...
```

**Layout:** `layouts/dashboard` (not `auth-entry` for dashboard features)

---

## 8. HTTP responses & errors

| Endpoint | Success | Form invalid | Service failure |
|---|---|---|---|
| ... | ... | ... | ... |

**Edge cases (≥5 for 🟡+ tasks):**

| # | Case | Behavior |
|---|---|---|
| 1 | ... | ... |

---

## 9. i18n

- Namespace: `...`
- User-facing: JA via `config/locales/ja.yml`

---

## 10. Security & tenant isolation

- Auth: ...
- Scope: `current_tenant_user` / `policy_scope`
- Cross-tenant prevention: ...

---

## 11. Observability

- AuditLog via services when `actor` + `request_id` passed
- Metrics/alerts: N/A or ...

---

## 12. Diagrams

### 12.1 Logical flowchart

```mermaid
flowchart TD
    ...
```

### 12.2 Sequence — happy path

```mermaid
sequenceDiagram
    ...
```

### 12.3 Sequence — error path

```mermaid
sequenceDiagram
    ...
```

---

## 13. Test design

| ID | Scenario | Expected |
|---|---|---|
| T1 | ... | ... |

| Layer | File(s) | Count |
|---|---|---|
| Request | `spec/requests/...` | ... |

---

## 14. Implementation order & rollback

1. ...
2. ...

**Rollback:** ...

---

## 15. Requirement mapping

| Requirement | DD section | Test |
|---|---|---|
| ... | §... | T... |

---

## Changelog

- YYYY-MM-DD — Created (DRAFT)
