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

### Contrast contract

Fill tokens (`--C-PRIMARY`, `--C-ACCENT`, status `bg-*`) are guaranteed to contrast only their _paired_ foreground (`--C-TEXT-ON-PRIMARY`, `--C-TEXT-ON-ACCENT`, …) — **never the surface**. A theme may legitimately set `--C-PRIMARY` ≈ `--C-SURFACE-*` (grimdark and tech both do). So:

- **Ink on a surface** — text, a 1–2px border, a connector line, an icon drawn directly on `--C-SURFACE-*` — must use a **text token** (`--C-TEXT-PRIMARY` / `-SECONDARY` / `-MUTED`); those are contract-guaranteed to read on the surface. Never a fill token.
- **A badge / chip / avatar filled with a fill token** must be outlined in its `on-*` text token, not in the fill itself. That ring is provably visible _exactly_ when the fill blends into the surface (if `fill ≈ surface`, then `on-fill` contrasts the surface too), and stays a quiet halo where the fill already pops.

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

**View transitions.** [src/animations/view-transitions.css](src/animations/view-transitions.css) wires `::view-transition-old(root)` / `::view-transition-new(root)` to `--MOTION-PAGE-TRANSITION-{OUT,IN}` (default `fade` / `fade-out`). To customise per theme, set those vars to your own `@keyframes` names — [the grimdark example theme](src/examples/themes/grimdark.css) does this.

### Media / overlay

Bracket-syntax escape hatch for `--media-*` and `--overlay-*` tokens — e.g. `aspect-[var(--media-aspect-poster)]`, `backdrop-blur-[var(--overlay-blur)]`.

## Theme switching

```html
<html data-theme="aurora">
  <!-- any name your own CSS defines -->
</html>
```

Omit `data-theme` for the default (`:root` token set, no override layer). Set the attribute however your app manages theme state — a `<script>` in `<head>`, a framework effect, whatever. No JS from this package required.

`default` is the only theme name this package has any opinion about. `events`,
`grimdark` and `tech` are worked examples under [src/examples/themes/](src/examples/themes/); nothing
imports them and nothing depends on them. Do not treat them as a fixed set, do
not add a fourth "official" theme, and do not write code or docs that enumerate
them — see **Don'ts**.

## Install contract

```css
@import "@batthewz/response-ui-css"; /* in consumer's app.css */
```

Pulls in, in order: the `default` theme's fonts → `tailwindcss` → tokens → responsive scales → animations → base. No PostCSS config needed. **No theme override layer is imported** — `default` is `:root`, and every other theme (examples included) is opt-in.

This package registers Tailwind sources only for its **own** files. Any separately-installed library that ships components — and the consumer's own source tree — must register its own `@source`. A sideways path into another package's folder (`@source "../../some-other-package/…"`) assumes npm's hoisted layout and silently matches nothing under isolated stores (bun, pnpm), so such a library must carry a **self-relative** `@source` in its own CSS entry; the consumer adds `@source "…"` directives for any sources outside their own tree.

### Subpath exports (rare — prefer the main entry)

The package also exposes these subpaths, for the narrow cases where the main entry is wrong:

| Subpath                                                   | When                                                                                                |
| --------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| `@batthewz/response-ui-css/no-fonts`                      | App self-hosts Poppins / Libertinus Mono                                                            |
| `@batthewz/response-ui-css/fonts`                         | Pull only the `default` theme's Google Fonts import (rare)                                          |
| `@batthewz/response-ui-css/examples/themes/<name>`        | Load a worked example (`events`, `grimdark`, `tech`) — demos and docs, not production               |
| `@batthewz/response-ui-css/theme-template`                | Programmatic access to the template (tooling)                                                       |
| `@batthewz/response-ui-css/tokens`                        | Tokens only, no responsive scales / animations / base (very rare; you usually want the whole entry) |

Default rule still applies: **use the main entry.** Subpaths exist for the listed edge cases, not as a menu.

## Where things live

| Path                                                | Purpose                                                                                                       |
| --------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| [src/index.css](src/index.css)                      | Public entry (with fonts)                                                                                     |
| [src/index-no-fonts.css](src/index-no-fonts.css)    | Public entry without fonts                                                                                    |
| [src/fonts.css](src/fonts.css)                      | Font imports only                                                                                             |
| [src/\_theme-template.css](src/_theme-template.css) | **Read this.** Theme contract with semantic comments                                                          |
| [src/tokens/](src/tokens/)                          | Default theme + `@theme inline` mappings (colors, radius, shadows, motion, transitions, media, overlay)       |
| [src/responsive/](src/responsive/)                  | `--R-SIZE-*`, `--H*`, `--BodyText-*` + Tailwind aliases                                                       |
| [src/animations/](src/animations/)                  | Keyframes + `.class` / `animate-*` utilities for fade, scale, morph, scroll-reveal, stagger, view-transitions |
| [src/examples/](src/examples/)                      | Worked example themes. Sample code — not imported by any entry, not covered by semver, safe to delete        |
| [src/base.css](src/base.css)                        | Root font setup, theme-aware scrollbars, `dialog[open].no-body-scroll` body-lock                              |

Strictly the design-system foundation. Ships no React components, no JavaScript.

## Authoring a custom theme

This is the normal case, not an advanced one — the example themes were written this way and get no privileges yours does not.

1. Copy [src/\_theme-template.css](src/_theme-template.css), change the `[data-theme="…"]` selector, replace the OKLCH values. Core coupled vars at top; optional ones (radius, shadow, motion, weights) at bottom — omit to inherit from `:root`.
2. `@import` it in `app.css` **after** `@import "@batthewz/response-ui-css";`.
3. Import your own font faces at the **top of the app's CSS entry**, above the foundation import — never inside the theme file. `@import` must precede all other rules in the flattened stylesheet, and a theme file is loaded after the foundation, so an `@import` there is dropped silently (correct palette, wrong typeface). The main entry loads only the `default` theme's two families; each example theme keeps its own in a sibling `<name>-fonts.css`.
4. Set `<html data-theme="your-theme">`.
5. If your theme is dark, or sets two contract tokens to the same colour, and you use `@batthewz/response-ui-react-components` charts, override `--C-CHART-1..5` too. Those tokens belong to that package — its `docs/theme-contract.md` holds the rule, not this package's.

If a theme overrides `--R-SIZE-*` / `--H*` / `--BodyText-*`, mirror the `@media (width >= 40rem)` block from [src/responsive/](src/responsive/) so the responsive bump survives.

## Don'ts

- No `tailwind.config.js` — v4 reads everything from CSS.
- No micro-imports — default to the main entry. Subpaths exist only for the narrow cases in the table above (self-hosted fonts, loading a worked example).
- No enumerating the example themes. Never write `events`/`grimdark`/`tech` into a type, a selector, a default value, a config list, or prose that implies a fixed set — outside `src/examples/` and the docs that explicitly frame them as examples. `scripts/verify-example-themes.mjs` in the react-components repo gates the equivalent rule there; the same rule applies here by convention.
- No Tailwind defaults (`p-4`, `text-sm`, `bg-blue-500`, `rounded`) — use tokens (`p-r3`, `text-body-2`, `bg-accent`, `rounded-md`).
- No unprefixed `text-primary` for foreground colour — collides with the type-scale utility under `tailwind-merge`. Use `text-fg-primary`.
- No incoherent partial themes. Mechanically a `[data-theme]` block only _overrides_, so any token you omit inherits `:root` and nothing is strictly required (see [docs/theme-contract.md](docs/theme-contract.md)). But the core set — `color-scheme`, canvas, surfaces, and the `--C-TEXT-*` ink that must contrast them — is **coupled**, and moving one without the others produces a provably broken theme (a dark canvas with default dark ink). Ship the core set together; radii, shadows, motion and weights are genuinely optional.
- No fill token (`--C-PRIMARY` / `--C-ACCENT` / status `bg-*`) as ink on a surface — it's only guaranteed to contrast its own `on-*` text, not the surface. Use a `--C-TEXT-*` token for ink/lines/borders, and outline filled chips in their `on-*` token. See **Contrast contract** above.
