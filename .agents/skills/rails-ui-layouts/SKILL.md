---
name: rails-ui-layouts
description: >-
  Rails + Hotwire UI layout patterns for authenticated dashboard vs standalone
  auth/error pages (ERB, Tailwind 4, Preline, BEM CSS). Use when adding or
  changing layouts, login/password-reset flows, 404 pages, flash toasts, auth
  CSS, or migrating Figma screens into Rails views. Reference implementation:
  ai-housemaker branch feat/ui-authentication-and-404.
---

# Skill: Rails UI Layouts (Dashboard + Auth Entry)

> **Source**: distilled from **48 commits** on `ai-housemaker` → `feat/ui-authentication-and-404` (Jun 2026).
> **Stack**: Rails 7.2, ERB, Hotwire, Tailwind 4, Preline, Stimulus, I18n (`:ja` default).

## When to use

- New page needs a **layout decision** (dashboard vs auth vs bare).
- Redesigning **login / password reset / 404** to match Figma.
- Adding **flash toasts** that work on mobile and desktop.
- Splitting **critical CSS** to avoid first-paint flash.

## Layout taxonomy

| Layout | Controller base | Use for |
|--------|-----------------|---------|
| `application` | Rare / mailers | Minimal shell, no chrome |
| `dashboard` | `BaseController` (`layout 'dashboard'`) | Authenticated tenant app: sidebar + header + main |
| `auth` | `SessionsController`, `PasswordsController`, `ErrorsController`, `render_not_found` | Login, password flows, 404 — centered card on branded background |
| `super_admin` | `SuperAdmin::BaseController` | Platform admin — top nav, no sidebar |

**Rule**: Do **not** create a separate `errors` layout. Reuse `auth` so 404 shares the same shell, background, and flash behavior.

## Auth layout architecture

```
layouts/auth.html.erb          ← shell only (head assets, background, flash, yield)
  └── page view                ← auth-entry DOM tree (main → panel → container → card)
        └── shared partials    ← brand header, form blocks, illustrations
```

### Shell (`layouts/auth.html.erb`)

- `body.auth-page` with `data-turbo="false"` on auth pages when full reload is safer.
- Fixed background via `.auth-page__background` (decorative, `aria-hidden`).
- Flash rendered **above** `yield` inside `.auth-page__content`.
- Stylesheet order (critical — do not reorder):
  1. `auth-critical` (ERB — inlined background URL, first paint)
  2. `application` (design tokens, Tailwind, `@tailwindcss/forms`)
  3. `auth-entry` (page structure + form overrides, unlayered to beat plugin defaults)

### Page DOM template (every auth page)

Reuse this skeleton; vary only the card interior and BEM modifiers:

```erb
<main class="auth-page__main auth-entry">
  <div class="auth-entry__panel">
    <div class="auth-entry__container">
      <%= render "shared/standalone_brand_header" %>  <%# omit on 404 if design says so %>

      <div class="auth-entry__card auth-entry__card--<variant>">
        <%# title, copy, form or CTA %>
      </div>
    </div>
  </div>
</main>
```

**Figma anchors** (keep in CSS comments + values):
- Card max width: `28rem` (448px)
- Brand → card gap: `2.5rem` (40px)
- Inner content width inside card: `24rem` (384px) for body copy blocks

### BEM naming

| Block | Elements / modifiers | Responsibility |
|-------|---------------------|----------------|
| `auth-page` | `__shell`, `__background`, `__content`, `__main` | Full-page shell |
| `auth-entry` | `__panel`, `__container`, `__card`, `__header`, `__label` | Centered entry flow |
| `auth-entry__card` | `--password-new`, `--password-edit`, `--password-sent`, `--password-success`, `--not-found` | Per-screen spacing/illustration |
| `auth-form` | `__field`, `__input`, `__submit`, `__errors` | Reusable form primitives |
| `auth-password-field` | `__toggle`, `__icon--show/--hide` | Password visibility (Preline `hs-toggle-password`) |

**Do**: extract repeated blocks into `app/views/shared/_*.html.erb` (e.g. `_standalone_brand_header`).
**Don't**: duplicate brand markup across 6+ views — one partial, locale-aware (SVG for JA, text spans for EN).

## CSS strategy

### Three-layer auth CSS

1. **`auth-critical.css.erb`** — only rules needed before `application.css` parses. Keep in sync with `auth.css` for `.auth-page` / `.auth-page__background`.
2. **`auth.css`** — imported into `application.css`; general auth tokens, legacy `.auth-card` if any.
3. **`auth-entry.css`** — linked only from `auth` layout; layout + form overrides **outside** `@layer` so they win over `@tailwindcss/forms`.

### Design tokens

Use CSS variables from `application.css` (`--ahm-color-*`, `--ahm-radius-sm`). Never hardcode one-off hex in views.

### Mobile

- Auth entry: `padding` on `__panel`, full-width card, test at `<640px`.
- Dashboard flash: in-flow on mobile (`px-6` matches `dashboard_page_shell`); `sm:absolute` overlay top-end on desktop.

## Dashboard layout (contrast)

- Tailwind utility-first in `layouts/dashboard.html.erb` (sidebar 260px, `lg:pl-[260px]`).
- `dashboard-layout` Stimulus: mobile sidebar drawer, backdrop, keyboard.
- Page content wraps with `dashboard_page_shell` helper — adjusts top padding when flash present.
- Modals: `yield :dashboard_modals` outside `<main>` scroll area.

Auth pages intentionally use **plain CSS files**, not Tailwind in views, for pixel-stable Figma match.

## Flash / toast system

Single partial `shared/_flash` with two modes:

| Mode | Trigger | Behavior |
|------|---------|----------|
| Default | `render "shared/flash"` in layout | Wrapper `#flash-messages`; iterates `flash` hash |
| `:toast` | Turbo Stream append | Single toast node with `data-controller="flash-toast"` |

Helper constants in `ApplicationHelper`:
- `TOAST_VARIANT_STYLES` — container, icon, dismiss, `max-w-[475px]`
- `toast_message_only?` — notice/success/alert show message only (no title row)
- `toast_aria_role` — `alert` vs `status`

**Placement**:
- Auth layout: flash inside shell, above page content.
- Dashboard: flash inside `<main>`, positioned via wrapper classes.

## Controller wiring

```ruby
class SessionsController < Devise::SessionsController
  layout "auth"
end

class PasswordsController < Devise::PasswordsController
  layout "auth"
  # Custom actions: sent, success — register routes outside Devise defaults
end

class ErrorsController < ActionController::Base
  layout "auth"
end

# In ApplicationController
def render_not_found
  respond_to do |format|
    format.html { render "errors/not_found", status: :not_found, layout: "auth" }
    format.any  { head :not_found }
  end
end
```

Password reset flow extras (from branch):
- Session keys: `password_reset_email`, `password_reset_sent_at`, `password_reset_resent`
- `RESEND_COOLDOWN = 5.minutes` with Stimulus `auth-resend-form` countdown
- Token expiry via `Devise.reset_password_within` (15 min) — surface in I18n copy
- `after_resetting_password_path_for` → dedicated success page (no auto sign-in)

## I18n

- Namespace: `auth.sessions`, `auth.passwords`, `auth.brand`, `errors.not_found`, `layout.flash`
- All user-facing strings in `config/locales/ja.yml` (+ `en.yml`)
- Specs: assert on **I18n keys**, not hardcoded Japanese (see `refactor(spec): remove locale + hard codes text`)

## Stimulus scope

Use Stimulus only for **time-varying or interactive** UI:
- `auth-resend-form` — cooldown timer, enable/disable resend button
- `flash-toast` — dismiss
- Preline data attributes for password toggle (`data-hs-toggle-password`)

Do **not** move static layout/spacing into JS.

## Assets checklist

| Asset | Path | Notes |
|-------|------|-------|
| Login background SVG | `app/assets/images/auth/login-background.svg` | Referenced in `auth-critical.css.erb` |
| Brand icon/text SVG | `app/assets/images/auth/brand-*.svg` | JA brand text as image |
| 404 illustration | `app/assets/images/errors/not-found-404.svg` | `alt=""`, `aria-hidden` on wrapper |

Register new stylesheets in `config/initializers/assets.rb` and `app/assets/config/manifest.js`.

## Implementation workflow

Copy this checklist per screen:

```
- [ ] Pick layout (auth / dashboard)
- [ ] Add route + controller action if non-Devise page
- [ ] Create view using auth-entry skeleton + card modifier
- [ ] Add I18n keys (JA + EN)
- [ ] Styles: modifier block in auth-entry.css (not inline Tailwind on auth pages)
- [ ] Extract shared partial if markup repeats ≥2 views
- [ ] Flash: redirect with `notice`/`alert` or render with errors
- [ ] Request spec: GET page, POST flow, flash assertions via I18n
- [ ] Visual: mobile + desktop, hard refresh (critical CSS)
```

## Commit convention (from branch)

```
feat(auth): <user-visible capability>
fix(auth): <visual/behavior fix>
style(auth): <CSS-only, no logic>
refactor(auth): <structure without visual change>
```

Scope `(auth)` for layout/auth-flow work; keep commits small (one concern per commit).

## Anti-patterns observed & avoided

| Avoid | Prefer |
|-------|--------|
| Separate `errors` layout | `layout "auth"` everywhere for entry-style pages |
| Tailwind classes in auth Figma pages | BEM + dedicated `auth-entry.css` |
| Hardcoded UI strings in ERB/specs | `t("auth....")` |
| Duplicated brand header in each view | `shared/standalone_brand_header` |
| Loading background only in `application.css` | `auth-critical.css.erb` first in `<head>` |
| Inline flash HTML per controller | `shared/_flash` + helper token map |

## Reference files (ai-housemaker)

| File | Role |
|------|------|
| `app/views/layouts/auth.html.erb` | Auth shell |
| `app/views/layouts/dashboard.html.erb` | App chrome |
| `app/assets/stylesheets/auth-entry.css` | Entry page CSS |
| `app/assets/stylesheets/auth-critical.css.erb` | Critical background |
| `app/views/shared/_flash.html.erb` | Toast partial |
| `app/views/shared/_standalone_brand_header.html.erb` | Brand block |
| `app/helpers/application_helper.rb` | `dashboard_page_shell`, toast helpers |
| `spec/requests/passwords_spec.rb` | Flow + flash examples |

## Related kit resources

- Stage 7 prompt: `.github/prompts/implement-ui-layout.prompt.md`
- PR titles/commits: `.agents/skills/pr-conventions/SKILL.md`
- i18n: user-facing JA via Rails I18n, never hardcode in views
