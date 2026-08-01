---
name: figma-svg-html-structure
description: >-
  Restructure existing Rails ERB/HTML to match the layer hierarchy in a Figma-exported
  SVG file. Use when the user provides or references a Figma SVG export, asks to align
  HTML structure with Figma layers, refactor markup to match design handoff structure,
  or map Figma group/layer names to BEM blocks and elements. Skip for greenfield UI from
  a live Figma URL (use implement-figma-design + MCP), icon-only flat SVGs, or CSS-only
  polish with no structural change.
paths:
  - "app/views/**/*.{erb,haml,slim}"
  - "app/assets/stylesheets/**/*.css"
  - "**/*.svg"
---

# Figma SVG → HTML structure

Align **DOM hierarchy** in existing views with a **Figma SVG export** used as the structural spec. Visual styling comes after structure matches; behavior (Stimulus, Turbo, forms) stays intact unless the user asks otherwise.

## When to use vs other skills

| Situation | Skill |
|-----------|--------|
| User has a **SVG file** (or paste) from Figma and wants HTML **restructured** to match | **This skill** |
| User shares **Figma URL** / desktop selection; build UI from scratch | `implement-figma-design` (MCP) |
| Only rename CSS classes / spacing tweaks; hierarchy already correct | BEM rule (`.cursor/rules/bem-css-html.mdc`) |

## Prerequisites

- Figma SVG for the **same frame/component** as the view being changed (whole frame export, not a flattened icon).
- Target ERB partial(s) and any feature CSS file (e.g. `app/assets/stylesheets/auth.css`).
- Read `.cursor/rules/bem-css-html.mdc` before editing markup + CSS (consuming apps may symlink under `.cursor/rules/quality/`).

## Workflow

Copy this checklist and track progress:

```
- [ ] Step 1: Obtain and read the Figma SVG
- [ ] Step 2: Build a structure tree (ignore drawing primitives)
- [ ] Step 3: Map layers → semantic HTML + BEM
- [ ] Step 4: Diff against current ERB
- [ ] Step 5: Restructure HTML (preserve behavior)
- [ ] Step 6: Sync CSS selectors with new classes
- [ ] Step 7: Self-check structure parity
```

### Step 1: Obtain the Figma SVG

Prefer **Export → SVG** on the target **Frame** or top-level **Component** (not “Copy as SVG” on a single vector unless that is the whole UI).

Save or read the file in-repo or from the user attachment. Confirm the root `id` matches the screen/component name in Figma.

### Step 2: Build a structure tree

Parse nested `<g id="...">` groups. **Ignore** for hierarchy mapping:

- `<path>`, `<rect>`, `<circle>`, `<line>`, `<defs>`, `<clipPath>`, `<mask>`, gradients
- Groups whose only job is clipping/masking (often unnamed or auto-generated ids)
- Duplicate wrapper groups Figma adds for effects (shadow/blur) when they have no design meaning

**Keep** groups that mirror layout regions: page shell, card, header, form row, footer, list item, badge, etc.

If the SVG is large, read top-level groups first; drill into one subtree at a time (same pattern as truncated Figma MCP fetches).

For parsing patterns and edge cases, see [references/figma-svg-patterns.md](references/figma-svg-patterns.md).

### Step 3: Map layers → semantic HTML + BEM

| Figma layer signal | HTML | BEM |
|--------------------|------|-----|
| Frame / auto-layout group (container) | `div`, `section`, `main`, `nav`, `form`, `ul`/`li` as appropriate | Block or `{block}__{element}` |
| Text layer (`<text>` in SVG) | `h1`–`h6`, `p`, `span`, `label` | Element under parent block |
| Image / vector icon (leaf, not layout) | `img`, `image_tag`, or existing `icon()` helper | `{block}__icon`, `{block}__image` |
| Button / CTA frame | `button` or `link_to` | `{block}__action` or `{feature}-button` |
| Input field frame | `input`, `textarea`, `select` | `{block}__input` (match existing form blocks) |

Naming rules (this repo):

- Derive kebab-case names from Figma layer names: `Auth Card / Header` → block `auth-card`, element `auth-card__header`.
- **One block per component**; no chained elements (`block__el__sub` ❌).
- Root wrapper = outermost meaningful frame in the SVG.
- User-visible copy stays in **I18n** — layer text in SVG is reference only, not hardcoded in ERB.

### Step 4: Diff against current ERB

Compare **nesting depth and sibling order**, not pixel positions:

- Missing or extra wrapper levels
- Wrong parent (e.g. title outside header group)
- Semantic tag mismatch (div vs button vs label)
- Classes that no longer reflect structure

List changes as a short tree diff before editing. Do not refactor unrelated partials.

### Step 5: Restructure HTML

- Match SVG group **order and nesting**; use semantic tags where the layer role is clear.
- Preserve: `data-*` (Stimulus), `form` actions, Turbo frames/streams, `link_to`/`button_to`, Pundit-visible records, accessibility (`aria-*`, labels).
- Replace generic wrappers (`__wrapper`, `__inner`) with names from Figma when the layer is named.
- Do **not** embed the Figma SVG inline as the page UI unless the user explicitly asks.

### Step 6: Sync CSS

- Update selectors in the feature CSS file in the **same change** as ERB (BEM rule).
- Move rules when elements move; avoid leaving orphan selectors.
- Prefer design tokens (`var(--ahm-color-*)`, `var(--text-primary)`) over copying fill/stroke from SVG.
- Use Tailwind only for one-off layout utilities; feature blocks belong in dedicated CSS.

### Step 7: Self-check

- [ ] Every meaningful `<g id="...">` in the SVG has a corresponding DOM node (or intentional merge documented in a brief English comment).
- [ ] Sibling order matches the SVG tree.
- [ ] BEM classes match CSS file selectors.
- [ ] No hardcoded user-facing strings added.
- [ ] Forms, links, and Stimulus controllers still work.

## Gotchas

- **Flattened export** — If the SVG has almost no `<g id="...">` groups, re-export the frame without “Flatten” / use a higher-level frame; this skill needs layer ids.
- **Sanitized ids** — Figma may change spaces, `/`, and duplicates (`Card`, `Card 2`). Match by hierarchy and screenshot, not id string alone.
- **Auto-generated ids** — `Rectangle 123`, `Group 456` carry no semantics; infer role from position and sibling text layers, or ask the user to rename layers in Figma and re-export.
- **Structure ≠ pixels** — SVG uses absolute coordinates; HTML uses flow/flex/grid. Match **tree shape**, then style with CSS.
- **Icons vs layout** — Single-path brand icons (e.g. `app/assets/images/auth/brand-icon.svg`) are assets, not page structure specs.

## Output

When reporting done, include:

1. **Structure tree** (3–8 lines): Figma layer → BEM → HTML tag
2. **Files changed**: ERB partial(s) + CSS
3. **Intentional deviations** (merged groups, semantic tag choices) in English

## Further reading

- [Cursor Agent Skills](https://cursor.com/docs/skills)
- [Skill creation best practices](https://agentskills.io/skill-creation/best-practices)
- [Optimizing skill descriptions](https://agentskills.io/skill-creation/optimizing-descriptions)
- BEM rule (kit canonical): `.cursor/rules/bem-css-html.mdc`
