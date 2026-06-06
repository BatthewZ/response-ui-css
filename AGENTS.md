# @batthewz/response-ui-css

## What this is

A pure-CSS design system. Tailwind v4, CSS-first (no `tailwind.config.js`). Tokens live as CSS custom properties on `:root`; themes override them under `:root[data-theme="…"]`. Tokens are re-exposed as Tailwind utilities via `@theme inline`. No JavaScript ships from this package.

## Why we want this

- **Unified theme.** One vocabulary for colour, type, spacing, motion, radius, shadow. No ad-hoc hex codes, no orphan `px` values.
- **Easy reskinning.** A theme is one CSS file overriding ~30 custom properties. Flip `<html data-theme="…">` and the whole app changes — no component edits, no rebuilds.
- **Responsive by default.** Spacing (`--R-SIZE-*`) and text (`--H*`, `--BodyText-*`) tokens carry a base value AND a `@media (width >= 40rem)` override — they scale at the 640px breakpoint for free. (Exception: `--R-SIZE-6` is intentionally flat at `0.25rem` across breakpoints — it's the "stays small" hairline.)

⚠️ If you're about to write a raw hex code, a raw `rem` value, or a Tailwind default like `p-4` / `text-sm` / `bg-blue-500` / `rounded` — stop. There is a token for it.

## How to use it

**1. Read [src/\_theme-template.css](src/_theme-template.css).** Source of truth for the theme contract — every required and optional custom property with semantic comments. Read it before writing CSS or generating utility classes.

**2. In raw CSS,** use `--C-*`, `--R-SIZE-*`, `--H*`, `--BodyText-*`, `--RADIUS-*`, `--SHADOW-*`, `--MOTION-*` directly. They pick up theme overrides and the responsive bump automatically.

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

**3. In inline Tailwind,** use the utilities below — exposed via `@theme inline` in [src/tokens/](src/tokens/) and [src/responsive/](src/responsive/). If your app uses `tailwind-merge`, extend its config for the `r1`–`r6` spacing, `h1`–`h6` / `body-1`–`3` text, and `fg-*` colour groups; otherwise conflicting utilities won't collapse.

### Colour

```
bg-canvas
bg-primary  bg-primary-hover  bg-primary-active
bg-secondary  bg-secondary-hover
bg-accent  bg-accent-hover
bg-surface-0  bg-surface-1  bg-surface-2  bg-surface-3
bg-status-error    bg-status-success    bg-status-warning    bg-status-info
bg-status-error-bg bg-status-success-bg bg-status-warning-bg bg-status-info-bg

text-fg-primary  text-fg-secondary  text-fg-muted
text-fg-inverse        (theme's opposite of body text — generic light-on-dark / dark-on-light)
text-fg-on-primary     (text on bg-primary / -hover / -active)
text-fg-on-accent      (text on bg-accent / -hover)
text-status-error  text-status-success  text-status-warning  text-status-info

border-border-default  border-border-strong  border-border-focus
ring-border-default    ring-border-focus
```

The colour palette is also addressable as unprefixed `bg-*` / `border-*` / `ring-*` / `text-*` (e.g. `border-primary`, `ring-accent`). **For foreground colour always use `text-fg-*`** — unprefixed `text-primary` collides with the typography scale under `tailwind-merge`. There's no `on-secondary` / `on-surface-*` / `on-status-bg` by design — those backgrounds are near-canvas by contract, so `text-fg-primary` reads on them.

### Typography (scales at 640px)

```
text-h1  text-h2  text-h3  text-h4  text-h5  text-h6
text-body-1  text-body-2  text-body-3
font-semibold  font-bold       (read from --Semibold-Weight / --Bold-Weight)
```

### Spacing (scales at 640px)

```
p-r1   p-r2   p-r3   p-r4   p-r5   p-r6      (also px-, py-, pt-, pr-, pb-, pl-)
m-r1   m-r2   m-r3   m-r4   m-r5   m-r6      (also mx-, my-, mt-, mr-, mb-, ml-)
gap-r1 gap-r2 gap-r3 gap-r4 gap-r5 gap-r6    (also gap-x-, gap-y-)
space-x-r1 … space-x-r6   space-y-r1 … space-y-r6
size-r1 … size-r6   w-r1 … w-r6   h-r1 … h-r6
top-r1 … top-r6   right-r1 … right-r6   bottom-r1 … bottom-r6   left-r1 … left-r6
inset-r1 … inset-r6   inset-x-r1 … inset-x-r6   inset-y-r1 … inset-y-r6
```

Any Tailwind utility taking a spacing value accepts `r1`–`r6` — the list above is illustrative, not exhaustive.

### Radius, shadow

```
rounded-sm  rounded-md  rounded-lg  rounded-xl  rounded-full
shadow-sm   shadow-md   shadow-lg
```

### Motion

```
duration-fast  duration-normal  duration-slow
duration-enter  duration-exit  duration-shift  duration-page
ease-enter  ease-exit  ease-shift  ease-page  ease-bounce
```

Additional motion tokens exposed for bracket-syntax use: `--motion-distance-{sm,md,lg}`, `--motion-stagger-delay`, `--motion-parallax-rate`, `--motion-scale-{hover,press}`. Use e.g. `translate-y-[var(--motion-distance-md)]`, `scale-[var(--motion-scale-hover)]`.

### Animations

**Before writing a `@keyframes`, check [src/animations/](src/animations/) — these are already provided.** Each ships as both a plain class (`.fade-in`) and a Tailwind utility (`animate-fade-in`), all duration/easing pulled from the motion tokens above, all with `prefers-reduced-motion` honoured.

```
animate-fade-in  animate-fade-down  animate-fade-up  animate-fade-left  animate-fade-right
animate-fade-out
animate-slide-in-right  animate-slide-out-right
animate-scale-in  animate-scale-out
animate-morph-expand  animate-morph-collapse  animate-morph-reorder
```

Non-utility helper classes (apply directly, no Tailwind utility):

- `.scroll-reveal-hidden` — starting opacity 0; pair with one of the `animate-*` utilities above when an IntersectionObserver flips visibility. Reduced-motion users skip the hidden state.
- `.stagger-item` — reads `--stagger-index` (set inline per child, e.g. `style={{ "--stagger-index": i }}`) and multiplies it by `--MOTION-STAGGER-DELAY`. Pair with any `animate-*` utility.

**View transitions.** [src/animations/view-transitions.css](src/animations/view-transitions.css) wires `::view-transition-old(root)` / `::view-transition-new(root)` to `--MOTION-PAGE-TRANSITION-{OUT,IN}` (default `fade` / `fade-out`). To customise per theme, set those vars to your own `@keyframes` names (see grimdark for an example).

### Media / overlay

Bracket-syntax escape hatch for `--media-*` and `--overlay-*` tokens — e.g. `aspect-[var(--media-aspect-poster)]`, `backdrop-blur-[var(--overlay-blur)]`.

## Theme switching

```html
<html data-theme="grimdark">   <!-- or "events", "tech" -->
```

Omit `data-theme` for the default (`:root` token set, no override layer). Set the attribute however your app manages theme state — a `<script>` in `<head>`, a framework effect, whatever. No JS from this package required.

## Install contract

```css
@import "@batthewz/response-ui-css";   /* in consumer's app.css */
```

Pulls in, in order: Google Fonts → `tailwindcss` → tokens → responsive scales → animations → built-in themes → base. No PostCSS config needed.

The entry already includes `@source` directives for `@batthewz/response-ui-react-components` (both `src/**/*.{ts,tsx}` and `dist/**/*.{js,mjs,cjs}`) — that sibling package is scanned out of the box. For any **other** separately-installed component library used outside the consumer's own source tree, add additional `@source "…"` directives after the import.

### Subpath exports (rare — prefer the main entry)

The package also exposes these subpaths, for the narrow cases where the main entry is wrong:

| Subpath | When |
| --- | --- |
| `@batthewz/response-ui-css/no-fonts` | App self-hosts Poppins / its theme fonts |
| `@batthewz/response-ui-css/fonts` | Pull only the Google Fonts import (rare) |
| `@batthewz/response-ui-css/themes/{events,grimdark,tech}` | Cherry-pick one built-in theme without loading the others |
| `@batthewz/response-ui-css/theme-template` | Programmatic access to the template (tooling) |
| `@batthewz/response-ui-css/tokens` | Tokens only, no responsive scales / animations / base (very rare; you usually want the whole entry) |

Default rule still applies: **use the main entry.** Subpaths exist for the listed edge cases, not as a menu.

## Where things live

| Path | Purpose |
| --- | --- |
| [src/index.css](src/index.css) | Public entry (with fonts); auto-`@source`s the React components package |
| [src/index-no-fonts.css](src/index-no-fonts.css) | Public entry without fonts |
| [src/fonts.css](src/fonts.css) | Font imports only |
| [src/\_theme-template.css](src/_theme-template.css) | **Read this.** Theme contract with semantic comments |
| [src/tokens/](src/tokens/) | Default theme + `@theme inline` mappings (colors, radius, shadows, motion, transitions, media, overlay) |
| [src/responsive/](src/responsive/) | `--R-SIZE-*`, `--H*`, `--BodyText-*` + Tailwind aliases |
| [src/animations/](src/animations/) | Keyframes + `.class` / `animate-*` utilities for fade, scale, morph, scroll-reveal, stagger, view-transitions |
| [src/themes/](src/themes/) | Built-in `events`, `grimdark`, `tech` overrides |
| [src/base.css](src/base.css) | Root font setup, theme-aware scrollbars, `dialog[open].no-body-scroll` body-lock |

Strictly the design-system foundation. Ships no React components, no JavaScript.

## Authoring a custom theme

1. Copy [src/\_theme-template.css](src/_theme-template.css), change the `[data-theme="…"]` selector, replace the OKLCH values. Required vars at top; optional ones (radius, shadow, motion, weights) at bottom — omit to inherit from `:root`.
2. `@import` it in `app.css` **after** `@import "@batthewz/response-ui-css";`.
3. Set `<html data-theme="your-theme">`.

If a theme overrides `--R-SIZE-*` / `--H*` / `--BodyText-*`, mirror the `@media (width >= 40rem)` block from [src/responsive/](src/responsive/) so the responsive bump survives.

## Don'ts

- No `tailwind.config.js` — v4 reads everything from CSS.
- No micro-imports — default to the main entry. Subpaths exist only for the narrow cases in the table above (self-hosted fonts, single-theme cherry-pick).
- No Tailwind defaults (`p-4`, `text-sm`, `bg-blue-500`, `rounded`) — use tokens (`p-r3`, `text-body-2`, `bg-accent`, `rounded-md`).
- No unprefixed `text-primary` for foreground colour — collides with the type-scale utility under `tailwind-merge`. Use `text-fg-primary`.
- No partial themes — all required colour / font / `color-scheme` variables must be present.
