# @batthewz/response-ui-css

## What this is

A pure-CSS design system. Tailwind v4, CSS-first (no `tailwind.config.js`). Tokens live as CSS custom properties on `:root`; themes override them under `:root[data-theme="…"]`. Tokens are re-exposed as Tailwind utilities via `@theme inline`. No JavaScript ships from this package.

## Why we want this

- **Unified theme.** One vocabulary of colour, type, spacing, motion, radius, shadow — every component in the app speaks it. No ad-hoc hex codes, no orphan `px` values.
- **Easy reskinning.** A theme is one CSS file overriding ~30 custom properties. Flip `<html data-theme="…">` and the whole app changes. No component edits, no rebuilds.
- **Responsive by default.** Spacing (`--R-SIZE-*`) and text (`--H*`, `--BodyText-*`) tokens carry a base value AND a `@media (width >= 40rem)` override. Using them means the component scales at the 640px breakpoint for free.

If you are about to write a raw hex code, a raw `rem` value, or a Tailwind utility like `p-4`/`text-sm`/`bg-blue-500` — stop. There is a token for it.

## How to use it

### 1. Read [src/\_theme-template.css](src/_theme-template.css)

That file is the source of truth for the theme contract: every required and optional custom property, with semantic comments. Read it before writing CSS or generating utility classes — it tells you what the design system actually models.

### 2. When writing CSS

Use the `--C-*`, `--R-SIZE-*`, `--H*`, `--BodyText-*`, `--RADIUS-*`, `--SHADOW-*`, `--MOTION-*` custom properties wherever they fit. Anything authored against these tokens automatically picks up theme overrides and the responsive breakpoint.

```css
.thing {
  background: var(--C-SURFACE-1);
  color: var(--C-TEXT-PRIMARY);
  padding: var(--R-SIZE-3);
  border: 1px solid var(--C-BORDER-DEFAULT);
  border-radius: var(--RADIUS-MD);
  transition: background var(--DURATION-FAST) var(--MOTION-EASE-ENTER);
}
```

### 3. When writing inline Tailwind classes

Every token below is exposed as a Tailwind utility via `@theme inline` blocks in [src/tokens/](src/tokens/) and [src/responsive/](src/responsive/). Prefer these over Tailwind defaults — they pick up theme overrides and the 640px responsive bump. If your consumer uses `tailwind-merge`, extend its config so it recognises the custom `r1`–`r6` spacing, `h1`–`h6` / `body-1`–`3` text, and `fg-*` colour groups; otherwise conflicting utilities won't collapse correctly.

**Colour — brand, surface, text, border, status**

```
bg-canvas
bg-primary  bg-primary-hover  bg-primary-active
bg-secondary  bg-secondary-hover
bg-accent  bg-accent-hover
bg-surface-0  bg-surface-1  bg-surface-2  bg-surface-3
bg-status-error  bg-status-error-bg
bg-status-success  bg-status-success-bg
bg-status-warning  bg-status-warning-bg
bg-status-info  bg-status-info-bg

text-fg-primary  text-fg-secondary  text-fg-muted  text-fg-inverse
text-fg-on-primary  text-fg-on-accent
text-status-error  text-status-success  text-status-warning  text-status-info
(also: text-primary, text-accent, text-canvas, text-surface-*  — same colour palette)

border-border-default  border-border-strong  border-border-focus
ring-border-default  ring-border-focus
(also: border-primary, border-accent, border-status-error, etc.)
```

Gotcha: text _colour_ uses the `fg-` prefix (`text-fg-primary`) — without it, `text-primary` resolves to the colour but collides with `text-h1`/`text-body-*` font-size utilities under `tailwind-merge`. Use `text-fg-*` for foreground colour and reserve unprefixed `text-*` for the typography scale below.

**Typography — responsive type scale (scales at 640px)**

```
text-h1  text-h2  text-h3  text-h4  text-h5  text-h6
text-body-1  text-body-2  text-body-3
font-semibold  font-bold   (use these — they read from --Semibold-Weight / --Bold-Weight)
```

**Spacing — responsive scale (scales at 640px)**

```
p-r1  p-r2  p-r3  p-r4  p-r5  p-r6     (and px-, py-, pt-, pr-, pb-, pl-)
m-r1 … m-r6                             (and mx-, my-, mt-, …)
gap-r1 … gap-r6                         (and gap-x-, gap-y-)
space-x-r1 … space-y-r6
size-r1 … size-r6, w-r*, h-r*
top-r*, left-r*, inset-r*, …
```

Any Tailwind utility that takes a spacing value accepts `r1`–`r6`.

**Radius, shadow**

```
rounded-sm  rounded-md  rounded-lg  rounded-xl  rounded-full
shadow-sm  shadow-md  shadow-lg
```

**Motion**

```
duration-fast  duration-normal  duration-slow
duration-enter  duration-exit  duration-shift  duration-page
ease-enter  ease-exit  ease-shift  ease-page  ease-bounce
```

**Media / overlay (escape-hatch arbitrary values)**

For card hover scale, scrim colour, overlay blur, aspect ratios, etc., the tokens are exposed as `--media-*` and `--overlay-*` — reach for them with bracket syntax when needed, e.g. `aspect-[var(--media-aspect-poster)]`, `backdrop-blur-[var(--overlay-blur)]`.

## Theme switching

```html
<html data-theme="grimdark">
  <!-- or "events", "tech" -->
</html>
```

Omitting `data-theme` gives the default theme (the `:root` token set, no override layer). Set the attribute however your app manages theme state — a `<script>` in `<head>`, a framework effect, whatever. No JS from this package is required.

## Install contract

```css
/* one import, in the consumer's app.css */
@import "@batthewz/response-ui-css";
```

That single import pulls in, in order: Google Fonts → `tailwindcss` → tokens → responsive scales → animations → built-in themes → base layer. No PostCSS config needed.

For Tailwind v4 to detect utility classes used outside the consumer's own source tree (e.g. in a separately-installed component library), add `@source "…"` directives in the consumer's `app.css` after the import.

## Where things live

| Path                                                       | Purpose                                                                                                 |
| ---------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| [src/index.css](src/index.css)                             | Public entry (with fonts)                                                                               |
| [src/index-no-fonts.css](src/index-no-fonts.css)           | Public entry without fonts                                                                              |
| [src/fonts.css](src/fonts.css)                             | Font imports only                                                                                       |
| [src/\_theme-template.css](src/_theme-template.css)        | **Read this.** Copyable template for custom themes — semantic comments on every variable                |
| [src/tokens/](src/tokens/)                                 | Default theme + `@theme inline` mappings (colors, radius, shadows, transitions, motion, overlay, media) |
| [src/responsive/](src/responsive/)                         | `--R-SIZE-*`, `--H*`, `--BodyText-*` and their Tailwind aliases                                         |
| [src/themes/](src/themes/)                                 | Built-in `events`, `grimdark`, `tech` overrides                                                         |
| [scripts/theme-from-json.mjs](scripts/theme-from-json.mjs) | CLI: JSON → CSS theme file                                                                              |

This package is strictly the design-system foundation: tokens, themes, responsive scales, animations, base styles. It ships no components and no JavaScript.

## Authoring a custom theme

1. Copy [src/\_theme-template.css](src/_theme-template.css), rename it, change the `[data-theme="…"]` selector, replace the OKLCH values. Required variables are at the top; optional ones (radius, shadow, motion, weights) at the bottom — omit to inherit from `:root`.
2. `@import` it in `app.css` **after** `@import "@batthewz/response-ui-css";`.
3. Set `<html data-theme="your-theme">` (or toggle it from whatever drives theme state in the consuming app).

If a theme overrides `--R-SIZE-*`, `--H*`, or `--BodyText-*`, mirror the `@media (width >= 40rem)` structure from [src/responsive/](src/responsive/) so the responsive bump survives.

## Don'ts

- Don't add a `tailwind.config.js` — v4 reads everything from CSS.
- Don't import individual token files; use the main entry.
- Don't use Tailwind defaults like `p-4`, `text-sm`, `bg-blue-500`, `rounded` — use `p-r3`, `text-body-2`, `bg-accent`, `rounded-md`.
- Don't use `text-primary` for foreground colour — that name collides with the type-scale utility under `tailwind-merge`. Use `text-fg-primary`.
- Don't omit required colour/font/`color-scheme` variables from a custom theme — consumers assume every required variable is present.
