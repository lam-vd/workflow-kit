# Figma SVG export — structure reference

Load this file when parsing a Figma SVG is ambiguous (sanitized ids, deep nesting, masks, or missing groups).

## Typical export shape

```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 480 640" fill="none">
  <g id="Auth Entry">
    <g id="Background">...</g>
    <g id="Main">
      <g id="Card">
        <g id="Header">
          <g id="Brand">...</g>
          <text id="Title">...</text>
        </g>
        <g id="Form">
          <g id="Field / Email">...</g>
        </g>
      </g>
    </g>
  </g>
</svg>
```

Map outer `g` → wrappers; `text` → copy placeholders (I18n in ERB); leaf vector groups → icons/images.

## Layer name → BEM

| Figma layer | BEM block/element |
|-------------|-------------------|
| `Auth Entry` | block `auth-entry` (page/screen root) |
| `Card` | `auth-entry__card` or block `auth-card` if shared |
| `Header` | `auth-card__header` |
| `Field / Email` | `auth-form__field` + `auth-form__input` |
| `Button / Primary` | `auth-form__submit` |

Rules:

- Strip variant suffixes into modifiers: `Card / Password sent` → `auth-entry__card--password-sent`.
- Slashes and spaces in ids are common; normalize to kebab-case.
- Prefer reusing **existing** blocks in the feature CSS (`auth-page`, `auth-card`, `auth-form`) over inventing parallel names.

## Groups to skip (not DOM nodes)

- `Clip path group`, `Mask group`
- ids matching `^Rectangle \d+$`, `^Ellipse \d+$`, `^Vector \d+$` with no child groups
- Effect-only wrappers (drop shadow) when a sibling group already defines the same region
- Empty `<g>` with only `<defs>`

## Groups to merge

- Multiple Figma groups that represent one logical row (icon + label + value) → one `{block}__row` with children
- Redundant auto-layout wrappers (group inside group, same bounds) → single element

## When ids are unreliable

1. Use **depth-first order** of remaining named groups.
2. Cross-check **text layer content** with locale keys (structure only).
3. Ask user to rename top-level frames in Figma and re-export — cheaper than guessing.

## Export settings (tell the user if structure is missing)

In Figma, export the **Frame** (not a slice):

- Format: SVG
- Do not flatten vectors if the option appears
- Include id attributes (default in Figma SVG export)

If the file is one giant `<path>` list, this reference cannot recover hierarchy — request a re-export.
