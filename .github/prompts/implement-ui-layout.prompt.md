---
mode: agent
description: "Stage 7 adjunct — Implement or redesign Rails UI layouts (dashboard shell, auth entry pages, flash toasts, 404). Use AFTER spec is FINAL when the sub-task touches ERB layouts, auth flows, or standalone entry screens. Loads .agents/skills/rails-ui-layouts/SKILL.md first."
---

You are implementing **Rails UI layout** work (auth entry, dashboard chrome, flash, error pages).

> **VI**: Triển khai layout UI Rails — trang auth/404, toast, hoặc dashboard shell. Chỉ chạy khi spec đã FINAL.

## Preconditions

1. Spec status = `FINAL`
2. Sub-task explicitly covers UI layout / auth screen / error page
3. If unclear which layout → STOP and ask

## Load context (in order)

```
1. docs/specs/<task>.md + docs/ddd/<task>.md   → screen requirements, copy, Figma refs
2. .agents/skills/rails-ui-layouts/SKILL.md      → layout patterns (MANDATORY)
3. .cursor/rules/clean-code.mdc
4. .agents/skills/pr-conventions/SKILL.md        → commit/PR format
```

## Decision tree: which layout?

```
Authenticated tenant feature (CRUD, lists)?
  → layout: dashboard
  → use dashboard_page_shell helper + Tailwind utilities
  → flash inside <main>

Login / password reset / invitation accept / 404?
  → layout: auth
  → auth-entry skeleton + auth-entry.css modifiers
  → NO separate errors layout

Platform super-admin?
  → layout: super_admin
```

## Implementation steps

### 1. Shell first
- Confirm or create `layouts/<name>.html.erb`
- Auth: verify stylesheet order `auth-critical` → `application` → `auth-entry`
- Register new CSS in `config/initializers/assets.rb` + manifest

### 2. Page view
- Use auth-entry DOM template from skill (main → panel → container → card)
- Add BEM modifier on card: `auth-entry__card--<screen>`
- Extract shared blocks to `app/views/shared/_*.html.erb` when duplicated

### 3. Styles
- Figma dimensions in CSS comments (448px card, 40px gap)
- Auth form rules **unlayered** in `auth-entry.css`
- Keep `auth-critical.css.erb` in sync with `auth.css` for background

### 4. Controller + routes
- `layout "auth"` on Devise controllers and `ErrorsController`
- Custom password pages: register GET routes (`sent`, `success`) + session state if needed
- `render_not_found` → `layout: "auth"`

### 5. I18n
- Keys under `auth.*`, `errors.*`, `layout.flash.*`
- JA + EN locale files
- Specs assert translation keys, not literal strings

### 6. Interactivity
- Stimulus only for cooldown, dismiss, toggles
- Preline `data-hs-toggle-password` for password fields

### 7. Tests
- Request specs: status, redirect chain, flash type/message
- Helper specs for toast class helpers if changed

## Output checklist

Before marking sub-task done:

- [ ] Correct layout assigned on controller
- [ ] No hardcoded user-facing text in ERB
- [ ] Mobile + desktop checked (auth panel padding, flash position)
- [ ] Hard refresh: no background/form flash (critical CSS)
- [ ] Shared partials used (no copy-paste brand header)
- [ ] Staged for review (`git add`) — commit in `/review-staged` when READY

## Impact tagging

| Change | Typical level |
|--------|---------------|
| CSS-only spacing/typography | 🟢 |
| New auth screen + routes | 🟡 |
| Flash contract change (dashboard + auth) | 🟠 |
| Auth security/session behavior | 🔴 — needs explicit spec + review |
