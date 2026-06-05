# AGENTS — @batthewz/response-ui-css

Machine-readable reference for AI assistants working with this package. Concise, exact, opinionated.

## What this is

A pure-CSS design system. Tailwind v4 (CSS-first config — no `tailwind.config.js`). All tokens live as CSS custom properties on `:root`; themes override them under `:root[data-theme="…"]`. Tokens are exposed to Tailwind utilities via `@theme inline`.

No JavaScript ships in this package.

## Install + import contract

```css
/* one import, in the consumer's app.css */
@import "@batthewz/response-ui-css";
```

That import internally does, in order:

1. Google Fonts for all four themes
2. `@import "tailwindcss";`
3. Tokens (`./tokens/index.css`) — colors, radius, shadows, transitions, motion, overlay, media
4. Responsive scales (`./responsive/index.css`) — spacing, text
5. Animation CSS (`./animations/index.css`) — fade, morph, scale, scroll-reveal, stagger, view-transitions
6. Theme overrides (`./themes/index.css`) — `events`, `grimdark`, `tech`
7. Base layer (`./base.css`) — root font, scrollbar styling, dialog body-lock
8. `@source "../../response-ui-react-components/src/**/*.{ts,tsx}"` and the same for `dist/**` so Tailwind v4's content-detection picks up classes used inside the React package

There is no PostCSS config to add. Tailwind v4 reads everything from CSS.

**Per-component CSS does NOT live here.** Accordion.css, AppShell.css, Button.css, etc. were moved into `@batthewz/response-ui-react-components` (co-located with each `.tsx`) and are imported via `@import "@batthewz/response-ui-react-components/styles"`. This package is strictly the **design-system foundation**: tokens, themes, responsive scales, animations, base styles. If you're adding a CSS file that is specifically the visual implementation of a React component, it belongs in the React package, not here.

## Theme switching

Themes are controlled by a single attribute:

```html
<html data-theme="grimdark">  <!-- or "events", "tech" -->
```

Removing `data-theme` (or omitting it) gives the `default` theme — that's just the `:root` token set, no override layer.

The React package's `useTheme()` hook is the canonical way to set this from JS, including localStorage persistence.

## The theme contract — required CSS variables

Anything defined as a custom property on `:root` is part of the contract. A custom theme MUST define the **required** ones; **optional** ones inherit from `:root` if omitted.

### Required — colors (OKLCH)

```
--C-CANVAS                 page background
--C-PRIMARY / -HOVER / -ACTIVE      brand primary
--C-SECONDARY / -HOVER              brand secondary
--C-ACCENT / -HOVER                  brand accent
--C-SURFACE-0 / -1 / -2 / -3         layered surfaces (cards, popovers)
--C-TEXT-PRIMARY / -SECONDARY / -MUTED / -INVERSE
--C-TEXT-ON-PRIMARY / -ON-ACCENT     text drawn on top of primary/accent fills
--C-BORDER-DEFAULT / -STRONG / -FOCUS
--C-STATUS-{ERROR,SUCCESS,WARNING,INFO}      foreground status colors
--C-STATUS-{ERROR,SUCCESS,WARNING,INFO}-BG   tinted backgrounds
```

### Required — typography

```
--DEFAULT-FONT             body font-family
--DEFAULT-MONO-FONT        monospace font-family
--HEADING-FONT             heading font-family (often = DEFAULT-FONT)
--HEADING-LETTER-SPACING   normal | <length>
--HEADING-TEXT-TRANSFORM   none | uppercase | lowercase
color-scheme: light | dark
```

### Optional — radius, shadow, motion, overlay, weights, text scales

If omitted, these inherit from `:root`. Override only what you want to change.

```
--RADIUS-{SM,MD,LG,XL,FULL}
--SHADOW-{SM,MD,LG}
--Bold-Weight, --Semibold-Weight
--MOTION-DURATION-{ENTER,EXIT,SHIFT,PAGE}
--MOTION-EASE-{PAGE,ENTER,EXIT,SHIFT,BOUNCE}
--MOTION-DISTANCE-{SM,MD,LG}
--MOTION-STAGGER-DELAY, --MOTION-PARALLAX-RATE
--MOTION-SCALE-{HOVER,PRESS}
--MOTION-PAGE-TRANSITION-{IN,OUT}        names of @keyframes
--MOTION-PAGE-{NEW,OLD}-ANIMATION-FILL-MODE
--OVERLAY-{SCRIM-COLOR, GRADIENT-START, GRADIENT-END, BLUR, BLUR-HEAVY}
--MEDIA-CARD-HOVER-{SCALE,LIFT}
--DURATION-{FAST,NORMAL,SLOW}
--H1..H6, --H1-line-height..H6-line-height
--BodyText-1..3, --BodyText-1-line-height..3-line-height
--R-SIZE-1..6                            responsive spacing scale
```

The `--R-SIZE-*`, `--H*`, and `--BodyText-*` tokens have BOTH a base value and a `@media (width >= 40rem)` override — they scale at the 640px breakpoint. If a custom theme overrides them, do so inside the same media-query structure.

### How tokens map to Tailwind utilities

`@theme inline` blocks in each token file expose Tailwind-friendly aliases. The mapping (relevant for AI generating utility classes):

```
--C-PRIMARY            → bg-primary, text-primary, border-primary, ring-primary
--C-PRIMARY-HOVER      → bg-primary-hover, ...
--C-CANVAS             → bg-canvas
--C-SURFACE-0..3       → bg-surface-0, bg-surface-1, ...
--C-TEXT-PRIMARY       → text-fg-primary       (note: "fg" prefix)
--C-TEXT-SECONDARY     → text-fg-secondary
--C-TEXT-MUTED         → text-fg-muted
--C-TEXT-INVERSE       → text-fg-inverse
--C-TEXT-ON-PRIMARY    → text-fg-on-primary
--C-TEXT-ON-ACCENT     → text-fg-on-accent
--C-BORDER-DEFAULT     → border-border-default, ring-border-default
--C-BORDER-STRONG      → border-border-strong
--C-BORDER-FOCUS       → border-border-focus, ring-border-focus
--C-STATUS-ERROR       → text-status-error, bg-status-error
--C-STATUS-ERROR-BG    → bg-status-error-bg
   (same pattern for SUCCESS, WARNING, INFO)

--R-SIZE-1..6          → p-r1, m-r1, gap-r1, ... (responsive spacing)
--H1..H6               → text-h1, text-h2, ...
--BodyText-1..3        → text-body-1, text-body-2, text-body-3

--RADIUS-SM..XL        → rounded-sm, rounded-md, rounded-lg, rounded-xl
--RADIUS-FULL          → rounded-full
--SHADOW-SM..LG        → shadow-sm, shadow-md, shadow-lg

--MOTION-DURATION-ENTER → duration-enter, etc.
--MOTION-EASE-ENTER     → ease-enter, etc.
--DURATION-FAST/NORMAL/SLOW → duration-fast, duration-normal, duration-slow
```

Important: when AI generates `cn(...)` calls in the React package, use the responsive `r1..r6` spacing and `h1..h6`/`body-1..3` text utilities instead of Tailwind defaults. The merge config in `@batthewz/response-ui-react-components/util/style` knows about these — using them ensures `tailwind-merge` collapses correctly.

## Authoring a new theme — minimal example

```css
:root[data-theme="aurora"] {
  color-scheme: dark;

  --DEFAULT-FONT: "Inter", sans-serif;
  --DEFAULT-MONO-FONT: "JetBrains Mono", monospace;
  --HEADING-FONT: "Inter", sans-serif;
  --HEADING-LETTER-SPACING: normal;
  --HEADING-TEXT-TRANSFORM: none;

  --C-CANVAS: oklch(0.18 0.04 270);
  --C-PRIMARY: oklch(0.6 0.15 220);
  --C-PRIMARY-HOVER: oklch(0.55 0.15 220);
  --C-PRIMARY-ACTIVE: oklch(0.5 0.15 220);
  --C-SECONDARY: oklch(0.3 0.05 220);
  --C-SECONDARY-HOVER: oklch(0.35 0.05 220);
  --C-ACCENT: oklch(0.7 0.2 90);
  --C-ACCENT-HOVER: oklch(0.65 0.2 90);

  --C-SURFACE-0: oklch(0.2 0.04 270);
  --C-SURFACE-1: oklch(0.23 0.04 270);
  --C-SURFACE-2: oklch(0.27 0.04 270);
  --C-SURFACE-3: oklch(0.31 0.04 270);

  --C-TEXT-PRIMARY: oklch(0.95 0.02 90);
  --C-TEXT-SECONDARY: oklch(0.75 0.02 90);
  --C-TEXT-MUTED: oklch(0.55 0.02 90);
  --C-TEXT-INVERSE: oklch(0.18 0.04 270);
  --C-TEXT-ON-PRIMARY: oklch(0.18 0.04 270);
  --C-TEXT-ON-ACCENT: oklch(0.18 0.04 270);

  --C-BORDER-DEFAULT: oklch(0.35 0.04 270);
  --C-BORDER-STRONG: oklch(0.45 0.04 270);
  --C-BORDER-FOCUS: oklch(0.7 0.2 90);

  --C-STATUS-ERROR: oklch(0.65 0.22 27);
  --C-STATUS-ERROR-BG: oklch(0.25 0.05 27);
  --C-STATUS-SUCCESS: oklch(0.7 0.18 145);
  --C-STATUS-SUCCESS-BG: oklch(0.25 0.05 145);
  --C-STATUS-WARNING: oklch(0.78 0.16 75);
  --C-STATUS-WARNING-BG: oklch(0.25 0.05 75);
  --C-STATUS-INFO: oklch(0.6 0.15 240);
  --C-STATUS-INFO-BG: oklch(0.25 0.05 240);
}
```

Don't forget to:
1. Tell `useTheme` about the new name: `useTheme({ themes: ["default", "aurora"] as const })`
2. `@import` the file in app.css **after** `@import "@batthewz/response-ui-css";`

## Files to remember

| Path | Purpose |
| --- | --- |
| `src/index.css` | Public entry (with fonts) |
| `src/index-no-fonts.css` | Public entry without fonts |
| `src/fonts.css` | Font imports only |
| `src/_theme-template.css` | Copyable template for custom themes |
| `src/tokens/colors.css` | The default theme's color tokens |
| `src/themes/{events,grimdark,tech}.css` | Built-in themes |
| `scripts/theme-from-json.mjs` | CLI: JSON → CSS theme file |

## Don'ts

- Don't add a `tailwind.config.js` — v4 reads everything from CSS.
- Don't import individual token files in app code unless you're explicitly opting out of the full system — use the main entry.
- Don't use Tailwind defaults like `p-4`, `text-sm` for the design system's responsive scale; use `p-r3`, `text-body-2` so themes can re-scale them.
- Don't define a theme's required color/font/`color-scheme` variables outside the contract — components rely on every required variable being present.
