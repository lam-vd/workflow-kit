# Repo gotchas — Figma ERB styling audit (example: ai-housemaker)

Optional reference for projects with shared modal/component CSS + design tokens.
Load when Step 3 (resolve effective CSS) shows unexpected values or global classes on the target element.
Adapt class/token names to the consuming app.

## CSS bundle import order

File: `app/assets/stylesheets/application.tailwind.css`

Feature-specific modal/detail CSS **must** be imported **after** `./components.css` so BEM overrides beat shared component classes.

```css
@import "./components.css";
@import "./house-options-edit-modal.css";  /* ✅ overrides ahm-modal-close-button */
@import "./house-series-edit-modal.css";
```

If a feature file is imported before `components.css`, shared rules win at equal specificity — symptoms include gray close icons, visible borders, and wrong hover states despite correct feature CSS.

## Shared classes that often conflict

| Class | Defined in | Typical conflict |
|-------|------------|------------------|
| `ahm-modal-close-button` | `components.css` | border, radius 8px, gray-400 color, hover background |
| `ahm-modal-button` | `components.css` | footer button sizing/colors |

Fix: feature `{block}__close-button` rules + correct import order. Do not remove shared classes from ERB unless the team standard changes — override in feature CSS instead.

## Design tokens (prefer over raw Figma hex)

Source: `app/assets/stylesheets/tokens.css`

| Figma common | Token |
|--------------|-------|
| `#1F2937` | `var(--ahm-color-text-heading)` |
| `#E5E7EB` | `var(--ahm-color-border)` |
| 18px title | `var(--ahm-font-size-lg)` |
| 600 weight | `var(--ahm-font-weight-semibold)` |

## Modal header pattern (reference)

Compare new work with `house-series-edit-modal.css`:

- Header height: `4.125rem`
- Title: `font-size-lg`, `font-weight-semibold`, `line-height: 1`, `letter-spacing: var(--ahm-letter-spacing-ui)`
- Close: 36×36, no border/radius, color `#1f2937`
- Header padding: asymmetric inline (`1.5625rem` start, `1.875rem` end)

## Icons

- Figma `Button Icons.svg` (36×36 X) maps to `icon(:modal_close_button, set: :custom_36, size: 36, "stroke-width": 1)`.
- Path definitions live in `app/helpers/icon_helper.rb` under `modal_close_button`.
- If paths match Figma but rendered icon looks wrong, suspect **CSS color/cascade**, not the helper.

## Typography from flat path SVG

When Figma exports title text as outlined paths (no `<text>`):

- Use SVG `viewBox` height as cap-height hint (e.g. 17px → `font-size-lg` 18px with `line-height: 1`).
- Do not infer weight from path thickness; use sibling modals or Figma inspect notes; default to `semibold` for modal titles in this repo.

## Tests

Styling-only fixes usually need **no new request specs**. Do not assert CSS classes in request specs (see `.cursor/rules/rspec-best-practices.mdc`).
