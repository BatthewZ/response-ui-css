# @batthewz/response-ui-css

CSS-first design tokens, themes, and component styles for the response-ui design system. Built on Tailwind CSS v4. Pairs with [`@batthewz/response-ui-react-components`](../response-ui-react-components/) but works on its own — useful from any framework, including server-rendered templates.

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

That's it. The package internally pulls in `tailwindcss`, all design tokens, all four built-in themes, responsive scales, animations, and component styles. There's no `tailwind.config.js`, no PostCSS config — Tailwind v4 is CSS-first.

If you self-host fonts and want to skip the Google Fonts imports:

```css
@import "@batthewz/response-ui-css/no-fonts";
```

## What ships

- **Design tokens** — colors (OKLCH), spacing, radii, shadows, motion, overlay, media query breakpoints
- **Responsive scales** — `--R-SIZE-1..6` spacing and `--H1..H6` / `--BodyText-1..3` text scales that step up at the 40rem (640px) breakpoint
- **Animations** — fade, scale, morph, scroll-reveal, stagger, view-transitions
- **Component styles** — backing CSS for AppShell, Accordion, Carousel, Popover, Tooltip, etc.
- **Four built-in themes** — switched via `<html data-theme="…">`:
  - `default` (the `:root` definitions; remove the attribute to apply)
  - `events` — warm editorial / celebratory
  - `grimdark` — gothic dark
  - `tech` — futuristic minimal

## Themes

Switch theme by setting `data-theme` on `<html>`:

```ts
document.documentElement.setAttribute("data-theme", "grimdark");
// or removeAttribute("data-theme") to revert to default
```

The React package exports a `useTheme()` hook that does this for you.

### Define your own theme

A theme is any selector that overrides the documented set of CSS custom properties. The convention is `:root[data-theme="…"]`. Three on-ramps:

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

2. **Generate from JSON** (e.g. exported by the ThemeEditor in the showcase):
   ```bash
   bunx @batthewz/response-ui-css theme-from-json my-theme.json --name aurora > src/themes/aurora.css
   ```

3. **Hand-write CSS** following the contract:
   ```css
   :root[data-theme="aurora"] {
     color-scheme: dark;
     --C-CANVAS: oklch(0.18 0.04 270);
     --C-PRIMARY: oklch(0.6 0.15 220);
     /* …rest of the contract — see docs/theme-contract.md */
   }
   ```

Full schema: [docs/theme-contract.md](./docs/theme-contract.md).

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
