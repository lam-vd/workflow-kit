---
name: figma-erb-styling-audit
description: >-
  Compare Rails ERB UI styling against Figma-exported SVG specs and apply minimal
  CSS or markup fixes only when visual differences are real. Use when the user
  @-mentions an ERB partial (often with line numbers) plus a Figma SVG file path,
  asks whether UI/styling matches Figma, or requests a styling update if different
  — including Vietnamese prompts like "khác với Figma", "cập nhật styling",
  "cập nhật lại nếu thật sự có sự khác nhau". Skip for DOM hierarchy refactors
  (figma-svg-html-structure), greenfield UI from a Figma URL (implement-figma-design),
  or explain-only questions with no fix intent.
paths:
  - "app/views/**/*.{erb,haml,slim}"
  - "app/assets/stylesheets/**/*.css"
  - "**/*.svg"
---

# Figma SVG → ERB styling audit

Compare **visual styling** of an existing Rails ERB region with a **Figma SVG export** of the same UI slice. Change code **only when a real mismatch exists** and the user grants implementation (follow the project's no-coding-without-permission rule when present).

## When to use vs other skills

| Situation | Skill |
|-----------|--------|
| User asks "does styling match Figma?" / "update if different" on ERB + SVG | **This skill** |
| User wants HTML **layer hierarchy** aligned to a grouped Figma SVG | `figma-svg-html-structure` |
| User shares **Figma URL** / builds UI from scratch | `implement-figma-design` |
| User only wants explanation, no code change | Read-only; do not apply this skill's fix steps |

## Typical user prompt shape

```
@app/views/feature/_partial.html.erb:7-12  @/path/to/FigmaLayer.svg  UI phần này đang có styling khác với Figma, cập nhật lại nếu thật sự có sự khác nhau.
```

Parse:

- **ERB target** — file path + optional line range (focus audit on that subtree).
- **Figma spec** — SVG export path (attachment or `@` reference).
- **Intent** — conditional fix: implement only if diff is confirmed.

## Workflow

Copy and track:

```
- [ ] Step 1: Read ERB slice + identify classes, helpers, global classes
- [ ] Step 2: Read Figma SVG + extract visual spec
- [ ] Step 3: Resolve effective CSS (feature file + globals + import order)
- [ ] Step 4: Build styling diff table
- [ ] Step 5: Decide — no change | CSS only | ERB + CSS
- [ ] Step 6: Apply minimal fix (if real diff + permission)
- [ ] Step 7: Report diff + files changed (or "already matches")
```

### Step 1: Read the ERB slice

- Open the partial and the **line range** the user cited.
- List every styling-relevant attribute:
  - `class` values (BEM block/element, shared components like `ahm-modal-close-button`)
  - Helpers: `icon(...)`, `image_tag`, inline `style=`
  - Parent wrappers that affect layout (flex header, modal shell)
- Note the **feature block** name (e.g. `house-options-edit`) for CSS lookup.

Do **not** change Stimulus `data-*`, form actions, Turbo frames, or I18n unless styling requires it.

### Step 2: Extract visual spec from the Figma SVG

Flat exports (common for handoff slices) are valid styling specs:

| SVG signal | Styling spec |
|------------|----------------|
| Root `width` / `height` / `viewBox` | Box size (e.g. 36×36 icon button) |
| `fill="#…"` on paths | Text/icon color |
| `stroke="#…"`, `stroke-width` | Icon stroke (default 1 if omitted) |
| `stroke-linecap` / `stroke-linejoin` | Icon geometry (should match `icon()` set) |
| Text as `<path>` (no `<text>`) | Typography slice — use viewBox **height** as cap-height hint (~font-size at `line-height: 1`) |
| Nested `<g id="…">` with rects | Padding, border, radius (when present) |

Ignore absolute coordinates for **flow layout**; use them for **sizes and colors** only.

For grouped frame exports, read only the subtree that matches the ERB slice.

### Step 3: Resolve effective CSS

1. Find feature CSS: `app/assets/stylesheets/{feature}*.css` matching the BEM block.
2. Trace **global overrides** on shared classes (`components.css`, `typography.css`).
3. Check **`application.tailwind.css` import order** — feature CSS must load **after** `components.css` when overriding shared modal/button styles. See [references/repo-gotchas.md](references/repo-gotchas.md).
4. For icons, compare Figma paths with `IconHelper` (`app/helpers/icon_helper.rb`, set e.g. `custom_36`).

**Reference implementation:** When auditing a modal slice, compare with the nearest sibling feature (e.g. `house-series-edit-modal.css` vs `house-options-edit-modal.css`).

### Step 4: Build a styling diff table

Before editing, write a short table:

| Property | Figma SVG | Current (effective) | Match? |
|----------|-----------|---------------------|--------|
| Font size | ~18px (17px bbox) | 20px xl | No |
| Font weight | semibold (600) | bold / invalid token | No |
| Icon color | #1F2937 | gray-400 (global win) | No |

Mark **Match?** conservatively. Token-equivalent values count as match (`#1f2937` ≡ `var(--ahm-color-text-heading)`).

### Step 5: Decide action

| Outcome | Action |
|---------|--------|
| All properties match (or token-equivalent) | **Stop.** Tell user UI already matches Figma; no diff. |
| Diff is from wrong CSS file / import order / missing override | Fix CSS (preferred). |
| Diff is from wrong `icon()` set, size, or `stroke-width` | Fix ERB helper args. |
| Diff needs new markup | Change ERB only if CSS cannot express the spec; keep BEM in sync. |

**Conditional fix rule:** If the user said "nếu thật sự có sự khác nhau" / "if really different", **do not edit** when the diff table is all matches.

### Step 6: Apply minimal fix

- **CSS:** Prefer design tokens (`var(--ahm-font-size-lg)`, `var(--ahm-color-text-heading)`) over copying hex from SVG when tokens align.
- **BEM:** Update HTML classes and CSS selectors in the **same change** (`.cursor/rules/bem-css-html.mdc`).
- **Scope:** Touch only the cited slice and its feature CSS; no drive-by refactors.
- **Icons:** Reuse existing icon helpers when paths already match Figma — fix CSS cascade instead of duplicating SVG inline.
- **I18n:** Do not hardcode copy from Figma text layers.

### Step 7: Report

**If changed:**

1. One-line verdict (what was wrong).
2. Diff table (before → after).
3. Files changed.

**If unchanged (all matches):**

1. One-line verdict: styling of the **cited ERB slice** already matches Figma.
2. Diff table for that slice only (all rows ✅).
3. `Files changed: none` (or omit if obvious).

**Response scope when unchanged — stay in slice:**

When the diff table is all matches, **do not expand the reply** beyond the cited ERB
lines / subtree and their effective CSS. In particular, **omit**:

- Cross-comparisons to sibling features, modal headers, or other screens not in the
  user's `@` ERB range (e.g. "khác modal title `#1F2937`").
- Reassurance about import order, cascade, or global CSS when no mismatch was found.
- Optional tweak offers for values that already match ("nếu muốn tăng lên `font-size-lg`…").
- A list of other UI areas worth auditing or follow-up slices to check.
- Next-step prompts unless the user asked for them.

Only mention another UI part when **that slice** has a confirmed Figma mismatch in the
same audit (same prompt's ERB + SVG pair), or the user explicitly asked about it.

**If unchanged — bad vs good report shape:**

```
❌ Match + ghi chú modal header khác màu, import order OK, "bạn muốn audit thêm X?"
✅ "Section title đã khớp Figma." + bảng diff (cited slice) + không đổi file.
```

## Gotchas (generic)

- **Shared component classes** — Global close/button classes often add border, radius, muted color, hover. Feature overrides need correct CSS **import order** (feature after shared).
- **Invalid / missing tokens** — Prefer defined design tokens; do not invent token names from Figma hex alone.
- **Flat text SVG** — Outlined text paths are typography specs, not structure specs; do not restructure HTML for them.
- **Match report brevity** — When all properties match, report only the cited slice (verdict + diff table). No adjacent UI notes, optional tweaks, or unsolicited audit backlog.
- **Permission** — Questions alone are read-only; "cập nhật", "Tiến hành", "implement" grant code changes.

Project-specific cascade notes (example: ai-housemaker): [references/repo-gotchas.md](references/repo-gotchas.md).

## Further reading

- [Cursor Agent Skills](https://cursor.com/docs/skills)
- [Skill creation best practices](https://agentskills.io/skill-creation/best-practices)
- [Optimizing skill descriptions](https://agentskills.io/skill-creation/optimizing-descriptions)
- Related: `.agents/skills/figma-svg-html-structure/SKILL.md`, `.cursor/rules/bem-css-html.mdc`
