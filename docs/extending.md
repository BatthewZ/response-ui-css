# Extending the foundation

This guide is for anyone building **on top of** `@batthewz/response-ui-css` — your own
components, your own utilities, your own themes — in any framework (Astro, Rails, Phoenix,
Svelte, Vue, plain HTML, or a React component library of your own).

This package is the **foundation layer**: design tokens, themes, responsive scales,
animations, and base styles. It ships **no JavaScript** and knows nothing about how you
build components. Everything below is pure CSS + Tailwind v4.

> Scope note: this document only covers the foundation. How you wire class-merging
> helpers or React components on top is the concern of the packages that depend on this
> one — not of the foundation itself.

---

## The mental model

There are two layers to every token:

1. **The raw value** — a CSS custom property on `:root`, e.g. `--C-PRIMARY`, `--R-SIZE-3`.
   Themes override these under `:root[data-theme="…"]`; responsive tokens carry a second
   value under `@media (width >= 40rem)`.
2. **The Tailwind utility** — generated from the raw value via an `@theme inline` block,
   e.g. `--color-primary: var(--C-PRIMARY)` is what makes `bg-primary` / `text-primary`
   exist.

Extending the system means adding to one or both layers, in your own CSS, after you import
the package.

---

## 1. Using existing tokens in your own components

**In raw CSS**, read the custom properties directly. They pick up theme overrides and the
responsive bump automatically:

```css
.my-card {
  background: var(--C-SURFACE-1);
  color: var(--C-TEXT-PRIMARY);
  padding: var(--R-SIZE-3);
  border: 1px solid var(--C-BORDER-DEFAULT);
  border-radius: var(--RADIUS-MD);
  transition: background var(--DURATION-FAST) var(--MOTION-EASE-ENTER);
}
```

**In markup**, use the generated Tailwind utilities (`bg-surface-1`, `text-fg-primary`,
`p-r3`, `text-h2`, `rounded-md`, …). The full utility list is in the
[README](../README.md).

### Respect the contrast contract

Fill tokens (`--C-PRIMARY`, `--C-ACCENT`, status `bg-*`) are guaranteed to contrast only
their **paired** foreground (`--C-TEXT-ON-PRIMARY`, etc.) — **never the surface**. A theme
may set `--C-PRIMARY ≈ --C-SURFACE-*`. So when you author your own components:

- **Ink on a surface** (text, a 1–2px border, a connector line, an icon drawn on
  `--C-SURFACE-*`) must use a **text token** (`--C-TEXT-PRIMARY/-SECONDARY/-MUTED`).
- **A chip/badge/avatar filled with a fill token** must be outlined in its `on-*` text
  token, never in the fill itself.

This is what keeps your components legible across every theme.

---

## 2. Adding your own tokens

When the built-in vocabulary isn't enough (a brand colour, a domain palette, a new
spacing step), extend the system rather than reaching for raw values.

### A universal token (theme-aware)

```css
/* your-tokens.css — imported AFTER @import "@batthewz/response-ui-css"; */

:root {
  --C-BRAND: oklch(0.62 0.19 250);
  --C-BRAND-HOVER: oklch(0.55 0.2 250);
}

/* Expose it to Tailwind so `bg-brand` / `text-brand` / `border-brand` exist */
@theme inline {
  --color-brand: var(--C-BRAND);
  --color-brand-hover: var(--C-BRAND-HOVER);
}
```

If a token should change per theme, override the raw value under each theme selector — the
`@theme inline` mapping does not need repeating:

```css
:root[data-theme="grimdark"] {
  --C-BRAND: oklch(0.7 0.16 250); /* lighter so it reads on dark surfaces */
}
```

### A responsive token

Responsive tokens carry **two** values. Define both breakpoints so the bump survives:

```css
:root {
  --R-SIZE-XWIDE: 4rem;
}
@media (width >= 40rem) {
  :root {
    --R-SIZE-XWIDE: 6rem;
  }
}
@theme inline {
  --spacing-rxwide: var(--R-SIZE-XWIDE); /* p-rxwide, gap-rxwide, … */
}
```

### Naming, so you don't collide

- Don't shadow the typography scale. Foreground colours use a `fg-` prefix
  (`text-fg-primary`) precisely so they don't collide with `text-h1` / `text-body-2`.
  Follow the same discipline for your own text colours.
- Mirror the existing token shape: `--C-*` for colour, `--R-SIZE-*` for spacing,
  `--RADIUS-*`, `--SHADOW-*`, `--MOTION-*`.

### Keep the foundation universal

A token earns a place in a _foundation_ layer only if it's genuinely universal. Domain
tokens — a chart palette, media-card hover feel, anything tied to a specific kind of UI —
belong in the layer that owns that UI, not in a shared foundation. This is the same
boundary discipline this package applies to itself.

---

## 3. Generating utilities for your own source

Tailwind v4 only generates utilities for classes it can see. When you write components in
your own project, register your source so your `p-r3` / `bg-brand` classes are emitted:

```css
@import "@batthewz/response-ui-css";
@import "./your-tokens.css";

@source "./components/**/*.{ts,tsx,html,vue,svelte}"; /* path relative to THIS css file */
```

Keep `@source` paths **self-relative** to your own CSS entry. Don't point sideways into
another package's `node_modules` folder — such paths assume a hoisted layout and silently
match nothing under isolated stores (bun, pnpm).

---

## 4. Custom themes

A theme is just a selector overriding the documented set of custom properties. Two
on-ramps:

```bash
# Copy the annotated template:
cp node_modules/@batthewz/response-ui-css/src/_theme-template.css ./src/themes/synapse.css
```

```css
/* app.css — import after the package so your overrides win */
@import "@batthewz/response-ui-css";
@import "./themes/synapse.css";
```

```css
/* themes/synapse.css */
:root[data-theme="synapse"] {
  color-scheme: dark;
  --C-CANVAS: oklch(0.18 0.04 270);
  --C-PRIMARY: oklch(0.6 0.15 220);
  /* …the rest of the contract */
}
```

Set `<html data-theme="synapse">` to activate. If your theme overrides any responsive
token (`--R-SIZE-*`, `--H*`, `--BodyText-*`), mirror **both** breakpoints.

The complete required/optional schema is in [docs/theme-contract.md](./theme-contract.md).

---

## Checklist

- [ ] Imported tokens (raw `var(--…)` or generated utilities), never raw hex / `rem` / Tailwind defaults
- [ ] Honoured the contrast contract (ink = text tokens; filled chips outlined in `on-*`)
- [ ] New tokens defined on `:root` **and** exposed via `@theme inline`
- [ ] Responsive tokens define both breakpoints
- [ ] Theme-varying tokens overridden under each `[data-theme]`
- [ ] Your own source registered with a self-relative `@source`
