# @batthewz/response-ui-css

The goal of this project: Reduce cognitive load for devs and AI by providing simple, standardized design tokens with sensible defaults that are overrideable. Building your project with the methodology of Response UI CSS will make it significantly easier to maintain over time.

While this is a pure CSS package, it is intended to be used with Tailwind v4 and integrates with it automatically.

### There are two complementary parts to this CSS approach:

1. Responsive design tokens for responsive [text](./src/responsive/text.css), [spacing](./src/responsive/spacing.css) - A single token represents different sizes across breakpoints.
2. An overridable [theme contract](./docs/theme-contract.md) that can be used as a basis for the style and branding of your page - Enough tokens to be comprehensive, but few enough to encourage consistency and maintainability.

> **Live demo:** [ai-website-starter.benmatthews-it.workers.dev/demo](https://ai-website-starter.benmatthews-it.workers.dev/demo) — every component, several themes, every responsive scale, in one place. This demo uses [Response UI React Components](https://www.npmjs.com/package/@batthewz/response-ui-react-components) which are built using Response UI CSS.

### Benefits of adopting this package:

- Standardizes theming and styling across your components, pages and mobile/web views
- Reduces time and cognitive load trying to get things looking right across viewports
- Makes re-skinning components or your entire app trivial
- Integrates with Tailwind v4 out of the box as overridable custom theme tokens
- Sensible defaults that are intentionally overridable and extendable
- Provides structure without inhibiting flexibility
- Includes [AGENTS.md](./AGENTS.md) for AI guidance on how to use and extend these tokens
- Significantly reduces AI token usage due to much lighter syntax
- Once integrated, change the entire feel of your page (everything from spacing, sizing and colours through to fonts, shadows, rounding and animation timing/easing) with only ~1 page of CSS.

**Example WITHOUT responsive tokens**

```html
<div className="flex flex-col gap-2 sm:gap-4">
  <h1
    className="text-lg sm:text-xl font-semibold sm:font-bold font-[My_Heading_Font]"
  >
    Some Heading
  </h1>
  <p className="text-md sm:text-lg">Some paragraph</p>
</div>
```

**Example WITH responsive tokens**

```html
<div className="flex flex-col gap-r3">
  <h1 className="text-h2 font-semibold">Some Heading</h1>
  <p className="text-body-1">Some paragraph</p>
</div>
```

---

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

That's it. The package internally pulls in `tailwindcss`, all design tokens, the default theme and its two fonts, responsive scales, animations, and base styles. If you're layering more CSS on top — a component library's styles, your own component CSS, or your own theme — import it **after** this package so it can read the tokens:

```css
@import "@batthewz/response-ui-css";
@import "./your-other-styles.css";
```

If you self-host fonts and want to skip the Google Fonts imports:

```css
@import "@batthewz/response-ui-css/no-fonts";
```

The full token list and guide is in [docs/theme-contract.md](./docs/theme-contract.md) and [docs/extending.md](./docs/extending.md) for how best to work with this package with your own components.

## The responsive scale

The `r` in utilities like `p-r3`, `gap-r4`, and the matching `text-h2` / `text-body-1` text scales stands for **Responsive**. One token, two breakpoints, no media queries in your markup: each value has a mobile-first base and an automatic step-up at `40rem` (640px). Write `gap-r3` once — it's `1rem` on mobile, `1.5rem` on desktop.

**One numbering rule across the whole system: `1` is the most-significant value, numbers ascend as values shrink.**

| Scale     | Utilities                                                                                     | Shape   | Notes                                                                                                                          |
| --------- | --------------------------------------------------------------------------------------------- | ------- | ------------------------------------------------------------------------------------------------------------------------------ |
| Headings  | `text-h1`..`text-h6`                                                                          | 6 sizes | Mirrors HTML `<h1>`..`<h6>` — apply the same number you'd put on the tag. The bare `<h1>`..`<h6>` elements are already styled. |
| Body text | `text-body-1`..`text-body-3`                                                                  | 3 sizes | Large / base / fine. Three is enough; more invites inconsistency.                                                              |
| Spacing   | `p-r1`..`p-r6`, `m-r1`..`m-r6`, `gap-r1`..`gap-r6` (and every other Tailwind spacing utility) | 6 sizes | `r1` = biggest gap, `r6` = tightest.                                                                                           |

Text scales come with **line-heights baked in** — every heading and body size carries its own paired line-height, and both the size _and_ the line-height step up at the desktop breakpoint. So `text-h2` on mobile uses a tighter leading sized for a smaller heading; on desktop the heading grows _and_ its leading opens up proportionally. You never set `leading-*` manually on responsive text — it's already correct at both breakpoints.

Headings and body text also **re-weight automatically**: `font-bold` and `font-semibold` step up by 100 at the desktop breakpoint, so smaller mobile text stays legible with a lighter weight, and larger desktop text gets the heavier weight it needs to feel balanced. You don't manage any of this — the tokens do.

**Why use these instead of Tailwind defaults like `p-4`, `text-sm`?** So themes can re-scale them. A theme that overrides `--R-SIZE-3` or `--H2` re-tunes every component that uses `p-r3` / `text-h2` in one shot — across both breakpoints.

Full token list and override surface: [docs/theme-contract.md](./docs/theme-contract.md).

## What ships

- **Design tokens** — colors (OKLCH), spacing, radii, shadows, motion, overlay, media query breakpoints
- **Responsive scales** — `--R-SIZE-1..6` spacing and `--H1..H6` / `--BodyText-1..3` text scales that step up at the 40rem (640px) breakpoint
- **Animations** — fade, scale, morph, scroll-reveal, stagger, view-transitions
- **Base styles** — resets, heading/body styling driven by the responsive type scale
- **The `default` theme** — the `:root` token set itself, plus its two font families
- **A theme contract and template** — everything you need to write your own, plus three worked examples that the package itself does not load

## Themes

**Theming is the point of this package. The themes it ships are not.**

A theme is one CSS file overriding ~30 custom properties under a `data-theme`
selector. Flip the attribute and the whole app re-skins — no rebuild, no
component edits. The design system has exactly one theme with any standing:
`default`, which _is_ `:root`. Everything else — including the three examples
below — is a consumer theme, and yours gets identical treatment.

### Write your own theme

Two on-ramps:

1. **Copy the template:**

   ```bash
   cp node_modules/@batthewz/response-ui-css/src/_theme-template.css ./src/themes/aurora.css
   # edit, then:
   ```

   ```css
   /* src/app.css */
   @import "@batthewz/response-ui-css";
   @import "./themes/aurora.css"; /* after the package, so it can override */
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

Then apply it, with no JavaScript from this package:

```html
<html data-theme="aurora"></html>
```

```ts
document.documentElement.setAttribute("data-theme", "aurora");
// removeAttribute("data-theme") returns to `default`
```

**Your theme brings its own fonts.** The main entry loads only the two families the
`default` theme names. Put your `@import url(...)` font lines at the very **top** of your
CSS entry — above the foundation import. CSS requires every `@import` to precede all other
rules in the flattened stylesheet, so a font import inside a theme file (which is loaded
_after_ the foundation) is silently dropped: the palette applies and the fonts never load.

```css
/* src/app.css */
@import url("https://fonts.googleapis.com/css2?family=Space+Grotesk&display=swap");
@import "@batthewz/response-ui-css";
@import "./themes/aurora.css";
```

Full schema and the required/optional split: [docs/theme-contract.md](./docs/theme-contract.md).

### The example themes

Three worked examples ship in `src/examples/themes/`. They exist to prove the
contract and to be read; **nothing in the design system depends on them, and no
package imports them.** Delete them from your `node_modules` and your app still
builds and renders — only this repo's own dev galleries, which deliberately demo
them, would notice. Treat them as sample code that happens to be installed, and expect them to
change or be replaced without ceremony.

| Example    | Shows                                                                        |
| ---------- | ---------------------------------------------------------------------------- |
| `events`   | A complete light theme — warm, editorial                                     |
| `grimdark` | A complete dark theme                                                        |
| `tech`     | The awkward case: two contract tokens sharing one colour                     |

To try one — fonts first, for the reason above:

```css
@import "@batthewz/response-ui-css/examples/themes/grimdark-fonts";
@import "@batthewz/response-ui-css";
@import "@batthewz/response-ui-css/examples/themes/grimdark";
```

They are **not** covered by this package's semver guarantees.

## Extending the foundation

Building your own components, utilities, or themes on top of these tokens — in any
framework? See [docs/extending.md](./docs/extending.md) for using and adding tokens (the
`:root` value + `@theme inline` pattern), responsive and theme-aware tokens, and
registering your own source with Tailwind.

## Subpath exports

| Export                                                | Use                                                                     |
| ----------------------------------------------------- | ----------------------------------------------------------------------- |
| `@batthewz/response-ui-css`                           | Main entry — Tailwind + tokens + `default` theme + base + animations    |
| `@batthewz/response-ui-css/no-fonts`                  | Same but without Google Fonts imports                                   |
| `@batthewz/response-ui-css/fonts`                     | Just the `default` theme's font imports                                 |
| `@batthewz/response-ui-css/tokens`                    | Tokens only — no Tailwind, no base styles                               |
| `@batthewz/response-ui-css/theme-template`            | The blank theme template — the starting point for your own              |
| `@batthewz/response-ui-css/examples/themes/<name>`    | A worked example (`events`, `grimdark`, `tech`). Sample code, not API — outside semver |

## License

MIT.
