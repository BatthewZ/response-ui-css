# @batthewz/response-ui-css

CSS-first design tokens, themes, responsive scales, and animations for the response-ui design system. Built on Tailwind CSS v4. The **foundation layer** — usable standalone from any framework (Astro, Phoenix, Rails, Svelte, Vue, plain HTML), or as the base for your own components and component libraries when you want the design system primitives without prescribed components.

> **Live demo:** [ai-website-starter.benmatthews-it.workers.dev/demo](https://ai-website-starter.benmatthews-it.workers.dev/demo) — every component, every theme, every responsive scale, in one place.

## Install

```bash
bun add @batthewz/response-ui-css
# peer dep, if you don't already have it:
bun add -D tailwindcss @tailwindcss/vite
```

## Use

One import in your app's CSS entry:

```css
/* src/app.css */
@import "@batthewz/response-ui-css";
```

That's it. The package internally pulls in `tailwindcss`, all design tokens, all four built-in themes, responsive scales, animations, and base styles. There's no `tailwind.config.js`, no PostCSS config — Tailwind v4 is CSS-first.

If you're layering more CSS on top — a component library's styles, your own component CSS, or custom themes — import it **after** this package so it can read the tokens:

```css
@import "@batthewz/response-ui-css";
@import "./your-other-styles.css";
```

If you self-host fonts and want to skip the Google Fonts imports:

```css
@import "@batthewz/response-ui-css/no-fonts";
```

## What ships

- **Design tokens** — colors (OKLCH), spacing, radii, shadows, motion, overlay, media query breakpoints
- **Responsive scales** — `--R-SIZE-1..6` spacing and `--H1..H6` / `--BodyText-1..3` text scales that step up at the 40rem (640px) breakpoint
- **Animations** — fade, scale, morph, scroll-reveal, stagger, view-transitions
- **Base styles** — resets, heading/body styling driven by the responsive type scale
- **Four built-in themes** — switched via `<html data-theme="…">`:
  - `default` (the `:root` definitions; remove the attribute to apply)
  - `events` — warm editorial / celebratory
  - `grimdark` — gothic dark
  - `tech` — futuristic minimal

## The responsive scale

The `r` in utilities like `p-r3`, `gap-r4`, and the matching `text-h2` / `text-body-1` text scales stands for **Responsive**. One token, two breakpoints, no media queries in your markup: each value has a mobile-first base and an automatic step-up at `40rem` (640px). Write `gap-r3` once — it's `1rem` on mobile, `1.5rem` on desktop.

**One numbering rule across the whole system: `1` is the most-significant value, numbers ascend as values shrink.**

| Scale | Utilities | Shape | Notes |
| --- | --- | --- | --- |
| Headings | `text-h1`..`text-h6` | 6 sizes | Mirrors HTML `<h1>`..`<h6>` — apply the same number you'd put on the tag. The bare `<h1>`..`<h6>` elements are already styled. |
| Body text | `text-body-1`..`text-body-3` | 3 sizes | Large / base / fine. Three is enough; more invites inconsistency. |
| Spacing | `p-r1`..`p-r6`, `m-r1`..`m-r6`, `gap-r1`..`gap-r6` (and every other Tailwind spacing utility) | 6 sizes | `r1` = biggest gap, `r6` = tightest. |

Text scales come with **line-heights baked in** — every heading and body size carries its own paired line-height, and both the size *and* the line-height step up at the desktop breakpoint. So `text-h2` on mobile uses a tighter leading sized for a smaller heading; on desktop the heading grows *and* its leading opens up proportionally. You never set `leading-*` manually on responsive text — it's already correct at both breakpoints.

Headings and body text also **re-weight automatically**: `font-bold` and `font-semibold` step up by 100 at the desktop breakpoint, so smaller mobile text stays legible with a lighter weight, and larger desktop text gets the heavier weight it needs to feel balanced. You don't manage any of this — the tokens do.

**Why use these instead of Tailwind defaults like `p-4`, `text-sm`?** So themes can re-scale them. A theme that overrides `--R-SIZE-3` or `--H2` re-tunes every component that uses `p-r3` / `text-h2` in one shot — across both breakpoints.

Full token list and override surface: [docs/theme-contract.md](./docs/theme-contract.md).

## Themes

Switch theme by setting `data-theme` on `<html>`:

```ts
document.documentElement.setAttribute("data-theme", "grimdark");
// or removeAttribute("data-theme") to revert to default
```

The React package exports a `useTheme()` hook that does this for you.

### Define your own theme

A theme is any selector that overrides the documented set of CSS custom properties. The convention is `:root[data-theme="…"]`. Two on-ramps:

1. **Copy the template:**
   ```bash
   cp node_modules/@batthewz/response-ui-css/src/_theme-template.css ./src/themes/aurora.css
   # edit, then:
   ```
   ```css
   /* src/app.css */
   @import "@batthewz/response-ui-css";
   @import "./themes/aurora.css";
   ```

2. **Hand-write CSS** following the contract:
   ```css
   :root[data-theme="aurora"] {
     color-scheme: dark;
     --C-CANVAS: oklch(0.18 0.04 270);
     --C-PRIMARY: oklch(0.6 0.15 220);
     /* …rest of the contract — see docs/theme-contract.md */
   }
   ```

Full schema: [docs/theme-contract.md](./docs/theme-contract.md).

## Extending the foundation

Building your own components, utilities, or themes on top of these tokens — in any
framework? See [docs/extending.md](./docs/extending.md) for using and adding tokens (the
`:root` value + `@theme inline` pattern), responsive and theme-aware tokens, and
registering your own source with Tailwind.

## Subpath exports

| Export | Use |
| --- | --- |
| `@batthewz/response-ui-css` | Main entry — Tailwind + tokens + themes + base + animations |
| `@batthewz/response-ui-css/no-fonts` | Same but without Google Fonts imports |
| `@batthewz/response-ui-css/fonts` | Just the font imports |
| `@batthewz/response-ui-css/tokens` | Tokens only — no themes, no Tailwind |
| `@batthewz/response-ui-css/themes/grimdark` | A specific theme file |
| `@batthewz/response-ui-css/themes/events` | … |
| `@batthewz/response-ui-css/themes/tech` | … |
| `@batthewz/response-ui-css/theme-template` | The blank theme template |

## License

MIT.
