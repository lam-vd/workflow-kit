---
name: writing-ddd-integrations
description: >-
  Stage 3 Detail Design checklist for external identity linking, file
  upload/download, retention vs link expiry, composite save-and-send, and
  adding a parallel integration without changing a live one. Use with
  writing-ddd when the feature touches OAuth/OIDC providers, consent gates,
  PHI/PII from third parties, chat or clinical attachments, signed URLs, or
  one-request multi-side-effect actions. Stack-agnostic wording.
---

# Writing DDD — integrations & attachments

## When to use

Load **after** `writing-ddd` when **any** of:

| Trigger | Examples (generic) |
|---------|-------------------|
| New / parallel external identity provider | OAuth/OIDC “login with X”, link/unlink account |
| Consent / opt-in before PII | Must call consent API before user-info / link |
| User-uploaded files in product surfaces | Chat media, clinical PDF slot, prescriptions |
| Signed / expiring download URLs | Short-lived GET links vs long-lived object storage |
| One user action → multiple side effects | “Send” = persist + call partner API in one request |
| Copy an existing integration path | Third provider beside two live ones |

Skip for pure CRUD UI with no external call and no file storage.

## How this skill is used

At `/write-spec` (and `/recheck-spec`): for each applicable **checklist ID** below, the DDD must have an explicit answer (or an Open Question / Decision Log entry). Missing answers → score down Security / Completeness, not “assume parity”.

Prefer **general terms** in the DDD (provider, consent flag, object store, signed URL). Product names belong in path citations and examples only.

---

## A. Current-state inventory (before design)

Document **as-is** before proposing **to-be**. Reviewers often ask these first.

### CUR-01 — Storage map

For every existing file type the feature extends or parallels:

| Field | Required |
|-------|----------|
| Store / backend | e.g. object store vs local |
| Bucket / container (or “from credentials”) | Env-specific names if known |
| Key prefix / namespace | Shared vs dedicated |
| Who may upload / download | Role + resource scope |

### CUR-02 — Retention vs link expiry (never conflate)

| Concept | Must state |
|---------|------------|
| **Object lifetime** | Keep forever? TTL? Soft-delete? |
| **Download-link lifetime** | Minutes/hours; regenerated on reload? |
| **Delete message / row** | Does it delete the object? |
| **Delete account / case** | Does it delete the object? |

If new media **inherits** existing policy, write that explicitly (“same as image chat: object kept; message hide only”).

### CUR-03 — Authz exceptions already in production

Call out intentional weak spots (e.g. “PDF download skips case-scoped auth so browser viewers work”). New types must say: **same exception** or **stricter** — never silent inherit.

---

## B. Parallel provider / copy-don’t-refactor

### PAR-01 — Do not mutate the live sibling

When adding a **third** (or Nth) parallel integration:

- **Copy** controller/service/table patterns from the closest sibling.
- **Do not** refactor the production path “while here” unless DoD requires it.
- State blast radius: which live flows stay untouched.

### PAR-02 — Separate persistence when uniqueness blocks multi-link

If the existing link table enforces **one row per account** (or one provider column), a second provider that must coexist needs a **dedicated table** (or a deliberate unique-index redesign + migration plan). Document **why** reuse was rejected (ADR-lite).

### PAR-03 — Shared domain hooks need regression tests

If one shared method (e.g. email-verify TTL, session cleanup) branches on “linked via provider X”:

- List **other** callers.
- Require regression cases for the default/legacy branch.

---

## C. Consent & PII from external identity

### CON-01 — Consent before retrieve

Order must be explicit:

1. Exchange token  
2. **Consent / opt-in check**  
3. Only if consented → fetch personal fields / link identity  

Document what happens if consent fails **after** token success (no link, no use of email/DOB).

### CON-02 — Dual-check on retrieve (race)

If both consent endpoint and user-info payload expose a consent flag: require **both** pass before linking or using PII. Name the error id.

### CON-03 — Unlink / consent revoked

| Event | Behavior |
|-------|----------|
| User unlinks in product settings | Delete link row; keep core account |
| Provider shows revoked consent on later visit | Destroy link (or equivalent); do not keep using tokens |
| App status API vs web `before_action` | If destroy is deferred, document **eventual** unlink vs immediate |

### CON-04 — No provider HTTP from model callbacks

Token refresh / consent / user-info run from **controller or orchestration service** only — not `after_save` / `after_commit` on the link model (avoids surprise outbound calls on unrelated saves).

---

## D. Files, download auth, retention (new types)

### FILE-01 — Dedicated slot vs shared chat/media store

When a file must **not** appear in chat attachment lists (or needs its own size/auth rules):

- Prefer **dedicated model + prefix/backend** over STI into the shared chat attachment type.
- Document UI surfaces that show / hide the file.

### FILE-02 — Auth lookup must match all stored types

If download middleware looks up by file id:

- Lookup must include **every** type that can be stored (base class / polymorphic), not only the first historical subclass.
- Missing lookup ⇒ signed URL alone may serve the file with **no** case/role check — call this out as a **must-fix** if adding a new type.

### FILE-03 — Download policy matrix

| Actor | Upload? | Download? | Inline vs attachment |
|-------|---------|-----------|----------------------|
| … | … | … | … |

Include “outsider / other org / unauthenticated → deny code”.

### FILE-04 — Signed URL rules

- Expiry duration  
- Who receives the URL (UI panel vs chat body)  
- **Never** persist expiring URLs in DB text columns  
- **Never** log signed URLs or tokens  
- Chat copy: filename + time OK; URL in chat text usually **no**

### FILE-05 — Replace / one-slot semantics

If “one file per parent”:

- New upload replaces prior  
- **When** old object is deleted (after successful DB commit preferred)  
- What user sees if partner send fails after replace  

### FILE-06 — MIME / size

- Server-side max size (single constant used by model + UI)  
- Allowlist MIME  
- State whether validation is **client-declared Content-Type only** (spoof risk) vs magic-byte — and whether that is acceptable for the audience (public vs staff-only)

### FILE-07 — Infra body / timeout limits

If max file size approaches reverse-proxy or app-server limits: raise proxy `client_max_body_size` (with small buffer) and document worker **read timeout** for slow uploads. Cite config paths.

---

## E. Composite actions (save + external send)

### CMP-01 — One button, ordered steps

Write the exact order, e.g.:

1. Authz  
2. Validate payload  
3. Consent / link preconditions  
4. Persist (if any)  
5. Call partner  
6. UX / chat side effects  

### CMP-02 — Fail-closed before persist

If consent, link, or authz fails → **no** object write and **no** partner call (unless Decision Log says otherwise).

### CMP-03 — Partner failure after persist

| Outcome | Persist | User messaging | Retry |
|---------|---------|----------------|-------|
| Partner 2xx | Keep | Success indicator | — |
| Partner error / timeout | Keep or roll back? | Error; no success chat | How? |

Default recommendation when doctor must retry: **keep** persisted file; do not emit “sent” chat on failure.

### CMP-04 — Sync vs job

If sync in one HTTP request: per-hop connect/read timeouts, disable double-submit, document what the UI shows while waiting. If job: idempotency key + status surface (usually a follow-up spec).

### CMP-05 — Contract-level TBD for partner APIs

Allowed in DRAFT/REVIEW when marked **CONTRACT-LEVEL + TBD**:

- Auth grant type, retry policy (e.g. 401 → refresh once), body shape  
- Explicit: host/credentials **not** guessed; filled when vendor delivers  

Not allowed as silent empty sections — use chapter “Contract + TBD” + Open Questions.

---

## F. Authz stricter than Ability alone

### AUTHZ-01 — Role gate + resource Ability

When a role (e.g. staff) inherits `update` on a resource but **UI must be doctor-only**:

- Document **both** checks (role predicate **and** `authorize` / policy).  
- Unauthorized → same status as sibling actions (often 404, not 403) if product hides existence.

### AUTHZ-02 — Panel data not returned to other roles

If JSON for the page includes file links: omit entirely for roles that must not see the panel (not only hide CSS).

---

## G. Security narrative for stakeholders

### SEC-01 — Do not claim “no security concern”

Stakeholder replies should separate:

| Layer | Example |
|-------|---------|
| **Access control (in scope)** | Who uploads/downloads; consent gate; short-lived links; size/MIME; force download |
| **Residual / policy (disclose)** | Object retention after account delete; MIME spoof parity; anyone with a live signed URL |

“Same as existing image/PDF policy” is valid **if** CUR-02/CUR-03 are filled.

### SEC-02 — Secrets hygiene

- Credentials in env/credentials samples as placeholders only  
- Never log access tokens, refresh tokens, Base64 file bodies, signed URLs  
- Error → Airbrake/monitor without those fields  

### SEC-03 — Mobile OAuth extras (when mobile in scope)

- `state` CSRF  
- `redirect_uri` allowlist  
- Session restore via one-time token header/param  
- Soft-fail when token expiry fields missing  
- Error redirects **without** issuing a login token  

---

## H. Spec quality gates (quick)

Before marking DDD ready for `/check-spec` on an integration feature:

- [ ] CUR-01…03 filled for touched file/identity surfaces  
- [ ] PAR-* if copying a live provider  
- [ ] CON-* if consent/PII  
- [ ] FILE-* if new downloadable type  
- [ ] CMP-* if save+send (or multi-effect) action  
- [ ] AUTHZ-* if UI role ⊂ Ability  
- [ ] SEC-01 wording usable with non-engineers  
- [ ] Known bugs / deferred behavior in Open Questions or “Pending scope” (not buried)  
- [ ] Test plan stubs external HTTP; asserts persist/auth; **no** CSS/Vue wiring in request specs  

---

## Anti-patterns

| Anti-pattern | Why |
|--------------|-----|
| “Link expires in 30m” without object retention | Stakeholders think files auto-delete |
| New file type + download lookup still on old subclass only | Auth skipped |
| Refactor live OAuth service to be “generic” in same PR | Cross-provider regression |
| Persist expiring URL in chat `text` | Dead links; leak via copy |
| “Security is fine” with no residual retention note | False confidence on PHI |
| Partner API hosts invented before vendor pack | Wrong env / secret sprawl |
| Model callback calls consent API | Hidden outbound I/O |
| Request specs assert CSS classes for panels | Brittle; out of layer |

## Related

- `writing-ddd` — base DDD structure  
- `shared-abstraction-safety` — no silent change to shared download/upload middleware  
- `edge-case-boundary-review` — size/MIME/consent boundaries  
- Project file-upload skills (e.g. Refile/ActiveStorage local skills) for implementation IDs  
