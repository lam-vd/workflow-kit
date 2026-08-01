---
name: css-safe-text-overflow
description: >-
  Prevent horizontal layout blowouts from long unbroken text (IDs, URLs, labels)
  and chip/tag rows in detail panels, modals, and label–value grids. Use when
  implementing or reviewing detail UI, definition lists, external IDs/URLs,
  chip groups, or when long content overflows a card/modal. Complements
  CSS-OVERFLOW-01 in hotwire-integration-patterns and INT-CSS-02 in
  integration-regression-review.
paths:
  - "app/assets/stylesheets/**/*.css"
  - "app/views/**/*.{erb,haml,slim,html}"
---

# Skill: Safe text overflow in constrained layouts

Keep **data mapping and business logic unchanged**. Overflow is a **CSS layout** concern: shrinkable tracks, wrap unbroken tokens, wrap chip groups.

## When to use

- Detail / modal / side panel with **label → value** rows (`dl` / grid).
- Values that may be **unbroken** (external IDs, URLs, codes, long Japanese compounds).
- Rows that render **chips / tags** (one or many).
- Review finds horizontal scroll or content pushing the page/modal wider.

## Failure modes

| Symptom | Common cause |
|---------|----------------|
| Page/modal grows horizontally | Value cell or ancestor missing `min-width: 0` |
| Long ID/URL stays on one line | Missing `overflow-wrap: anywhere` (or `word-break: break-word`) |
| Chip row overflows | Container not `flex-wrap`; chips lack `max-width: 100%` |
| Style “not applying” in browser | Edited source CSS but compiled asset not rebuilt |
| Modifier class loses to base rule | Specificity: `block element` beats a single class — use `block element.modifier` |

## Checklist (implement or review)

```
- [ ] Value column / cell: min-width: 0
- [ ] Shrinkable ancestors: flex/grid children + tracks (prefer minmax(0, …))
- [ ] Unbroken text: overflow-wrap: anywhere (or word-break: break-word)
- [ ] Links: wrap on the link or its cell (do not rely on parent alone)
- [ ] Chip groups: flex-wrap + gap; chips max-width: 100%; allow min-height grow if text wraps
- [ ] Narrow breakpoints: stack label above value when two-column grid is tight
- [ ] No logic change to “fix” overflow (do not truncate in Ruby/JS unless product asks)
- [ ] If CSS is compiled (e.g. Tailwind build): rebuild artifact after source edit
```

## Preferred CSS pattern

```css
.detail-card {
  min-width: 0;
}

.detail-card__row {
  display: grid;
  grid-template-columns: minmax(8rem, 1fr) minmax(0, 3fr);
  min-width: 0;
}

.detail-card__value {
  min-width: 0;
  overflow-wrap: anywhere;
}

.detail-card__value:has(> .chip) {
  display: flex;
  flex-wrap: wrap;
  gap: 0.375rem;
  align-items: center;
}

.chip {
  box-sizing: border-box;
  max-width: 100%;
  min-height: /* design token */;
  height: auto;
  overflow-wrap: anywhere;
  white-space: normal;
}
```

Adapt class names to the project’s BEM / design system. Keep fixed chip **height** only when labels are short; for long labels prefer `min-height` + wrap.

## Manual QA

1. Inject a long unbroken ID (~100+ chars) into the value cell → must wrap inside the card.
2. Inject a long URL → must wrap; link remains clickable.
3. Inject many chips or one very long chip → chips wrap to next line; no horizontal page scroll.
4. Resize to mobile width → label/value stack (if designed); still no horizontal scroll.
5. Hard refresh after CSS rebuild so the browser is not on a stale bundle.

## Related

- Rule catalog: `.cursor/rules/hotwire-integration-patterns.mdc` → **CSS-OVERFLOW-01**
- Stage 8 sweep: `.agents/skills/integration-regression-review/SKILL.md` → **INT-CSS-02**
- Do **not** encode product-specific field names into this skill — keep patterns reusable across apps.
