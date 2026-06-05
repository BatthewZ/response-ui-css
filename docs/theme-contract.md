# Theme contract

A theme is a CSS rule that overrides design-system tokens under a `data-theme` selector. The contract below is the authoritative list — any custom theme MUST define the **required** variables; **optional** ones inherit from `:root` if omitted.

> Selector convention: `:root[data-theme="<name>"]`. The `default` theme IS `:root` itself (no override layer); switching to `default` removes the `data-theme` attribute.

---

## Required

### Color scheme

```css
color-scheme: light | dark;
```

Drives form-control colors, default scrollbar appearance, etc. Set this — don't omit it.

### Brand colors

| Variable | Notes |
| --- | --- |
| `--C-CANVAS` | Page background |
| `--C-PRIMARY` | Brand primary fill |
| `--C-PRIMARY-HOVER` | Primary hover state |
| `--C-PRIMARY-ACTIVE` | Primary pressed state |
| `--C-SECONDARY` | Secondary fill |
| `--C-SECONDARY-HOVER` | Secondary hover state |
| `--C-ACCENT` | Brand accent (e.g. links, focus indicators) |
| `--C-ACCENT-HOVER` | Accent hover state |

All colors are OKLCH. Use OKLCH in your custom theme.

### Surfaces (layered backgrounds)

| Variable | Use |
| --- | --- |
| `--C-SURFACE-0` | Most foreground / most-elevated (usually on top of a popover scrim) |
| `--C-SURFACE-1` | Cards, navbar |
| `--C-SURFACE-2` | Slightly deeper containers |
| `--C-SURFACE-3` | Deepest container, often used for inputs/recessed regions |

For light themes these typically run from white → light gray. For dark themes, dark → slightly lighter shades.

### Text

| Variable | Use |
| --- | --- |
| `--C-TEXT-PRIMARY` | Default body text |
| `--C-TEXT-SECONDARY` | De-emphasized text (captions, helpers) |
| `--C-TEXT-MUTED` | Most-muted (placeholders, hints) |
| `--C-TEXT-INVERSE` | Text on a dark surface in a light theme (and vice versa) |
| `--C-TEXT-ON-PRIMARY` | Text drawn on `--C-PRIMARY` fill |
| `--C-TEXT-ON-ACCENT` | Text drawn on `--C-ACCENT` fill |

### Borders

| Variable | Use |
| --- | --- |
| `--C-BORDER-DEFAULT` | Default border (cards, inputs) |
| `--C-BORDER-STRONG` | Higher-contrast border |
| `--C-BORDER-FOCUS` | Focus ring color |

### Status

Each status has a foreground color and a tinted background:

| Foreground | Background |
| --- | --- |
| `--C-STATUS-ERROR` | `--C-STATUS-ERROR-BG` |
| `--C-STATUS-SUCCESS` | `--C-STATUS-SUCCESS-BG` |
| `--C-STATUS-WARNING` | `--C-STATUS-WARNING-BG` |
| `--C-STATUS-INFO` | `--C-STATUS-INFO-BG` |

### Typography

| Variable | Notes |
| --- | --- |
| `--DEFAULT-FONT` | Body font-family |
| `--DEFAULT-MONO-FONT` | Monospace font-family |
| `--HEADING-FONT` | Heading font-family (often = `--DEFAULT-FONT`) |
| `--HEADING-LETTER-SPACING` | `normal` or a `<length>` like `0.06em` |
| `--HEADING-TEXT-TRANSFORM` | `none` / `uppercase` / `lowercase` |

If you use a font that's not already loaded by `@batthewz/response-ui-css`, import the font-face yourself before your theme CSS.

---

## Optional (inherit from `:root` if omitted)

Override only what you want to change.

### Radius

```
--RADIUS-SM, --RADIUS-MD, --RADIUS-LG, --RADIUS-XL, --RADIUS-FULL
```

Defaults: `0.25rem / 0.5rem / 0.75rem / 1rem / 9999px`. Map to `rounded-sm`, `rounded-md`, etc. via `@theme inline`.

### Shadows

```
--SHADOW-SM, --SHADOW-MD, --SHADOW-LG
```

Map to `shadow-sm`, `shadow-md`, `shadow-lg`. For dark themes you'll often want deeper, less-blurry shadows.

### Responsive scales — shared convention

Three token families share one numbering rule and one breakpoint:

- **`1` is always the most significant value; numbers ascend as values shrink.**
- Each token has a **mobile-first base** and an **automatic step-up at `@media (width >= 40rem)`** (640px). That's where the `r` / "responsive" naming on the utilities comes from.
- If you override any of these tokens in a theme, override **both** breakpoints in the same media-query structure as [`src/responsive/text.css`](../src/responsive/text.css) and [`src/responsive/spacing.css`](../src/responsive/spacing.css).

| Family | Tokens | Count | Notes |
| --- | --- | --- | --- |
| Headings | `--H1, --H1-line-height` … `--H6, --H6-line-height` | 6 | Numbering mirrors HTML `<h1>`–`<h6>`, so `--H2` is what's applied to `<h2>` (and to `.h2` / `text-h2`). |
| Body text | `--BodyText-1, --BodyText-1-line-height` … `--BodyText-3, --BodyText-3-line-height` | 3 | Large / base / fine. Drives `text-body-1`..`text-body-3`. |
| Spacing | `--R-SIZE-1` … `--R-SIZE-6` | 6 | `R-SIZE-1` is the largest gap, `R-SIZE-6` the tightest. Drives `p-r1`..`p-r6`, `m-r1`..`m-r6`, `gap-r1`..`gap-r6`, and every other Tailwind spacing utility. |

### Line heights — paired with every size, also responsive

Every heading and body-text token has a matching `*-line-height` token (e.g. `--H2` ↔ `--H2-line-height`, `--BodyText-1` ↔ `--BodyText-1-line-height`). Two consequences:

1. **Each size carries its own leading.** A larger heading uses a more generous line-height; smaller body text uses tighter leading appropriate to its size. You don't pair `text-h2` with a `leading-*` utility — the line-height is already correct.
2. **Line-heights step up with sizes at 40rem.** When `--H2` grows from `1.75rem` to `3rem` at the desktop breakpoint, `--H2-line-height` grows from `2.25rem` to `4rem` in lockstep. The size/leading ratio is tuned per breakpoint, not a fixed multiplier.

If you override a font-size token, override its line-height counterpart at the **same breakpoint** — otherwise you'll get a desktop size at a mobile leading (or vice versa).

### Typography weights — auto-adjusted across breakpoints

```
--Semibold-Weight, --Bold-Weight
```

These also step at 40rem. The defaults are `500/600` on mobile and `600/700` on desktop — so small mobile text uses a lighter weight that reads cleanly at small sizes, and larger desktop text gets a heavier weight that holds visual hierarchy. They're plumbed into Tailwind as `--font-weight-semibold` / `--font-weight-bold`, so `font-semibold` and `font-bold` follow the breakpoint automatically. If you override them, override at both breakpoints.

### Motion

```
--MOTION-DURATION-{ENTER,EXIT,SHIFT,PAGE}
--MOTION-EASE-{PAGE,ENTER,EXIT,SHIFT,BOUNCE}
--MOTION-DISTANCE-{SM,MD,LG}
--MOTION-STAGGER-DELAY
--MOTION-PARALLAX-RATE
--MOTION-SCALE-{HOVER,PRESS}
--MOTION-PAGE-TRANSITION-IN, --MOTION-PAGE-TRANSITION-OUT  (names of @keyframes you define)
--MOTION-PAGE-NEW-ANIMATION-FILL-MODE, --MOTION-PAGE-OLD-ANIMATION-FILL-MODE
```

If you set `--MOTION-PAGE-TRANSITION-IN` / `OUT`, you also need to define the named `@keyframes` in your theme file.

### Transitions

```
--DURATION-FAST, --DURATION-NORMAL, --DURATION-SLOW
```

Map to `duration-fast`, `duration-normal`, `duration-slow`.

### Overlay

```
--OVERLAY-SCRIM-COLOR
--OVERLAY-GRADIENT-START, --OVERLAY-GRADIENT-END
--OVERLAY-BLUR, --OVERLAY-BLUR-HEAVY
```

Used by Spotlight, Carousel overlays, modal scrims.

### Media

```
--MEDIA-CARD-HOVER-SCALE, --MEDIA-CARD-HOVER-LIFT
```

Used by `MediaCard`. `SCALE` is a unit-less number (e.g. `1.02`); `LIFT` is a length (e.g. `-0.125rem`).

---

## Tailwind utility mapping

`@theme inline` blocks expose tokens to Tailwind utilities. The mapping follows a `--SCREAMING-CASE-FOR-TOKEN` → `lowercase-kebab-for-utility` convention with one wrinkle: `--C-TEXT-*` → `text-fg-*`.

| Token | Utility |
| --- | --- |
| `--C-CANVAS` | `bg-canvas` |
| `--C-PRIMARY` (and `-HOVER`, `-ACTIVE`) | `bg-primary`, `bg-primary-hover`, `bg-primary-active` (also `text-`, `border-`, `ring-`) |
| `--C-SECONDARY` (and `-HOVER`) | `bg-secondary`, `bg-secondary-hover` |
| `--C-ACCENT` (and `-HOVER`) | `bg-accent`, `bg-accent-hover` |
| `--C-SURFACE-0..3` | `bg-surface-0`, `bg-surface-1`, `bg-surface-2`, `bg-surface-3` |
| `--C-TEXT-PRIMARY` | `text-fg-primary` |
| `--C-TEXT-SECONDARY` | `text-fg-secondary` |
| `--C-TEXT-MUTED` | `text-fg-muted` |
| `--C-TEXT-INVERSE` | `text-fg-inverse` |
| `--C-TEXT-ON-PRIMARY` | `text-fg-on-primary` |
| `--C-TEXT-ON-ACCENT` | `text-fg-on-accent` |
| `--C-BORDER-DEFAULT` | `border-border-default`, `ring-border-default` |
| `--C-BORDER-STRONG` | `border-border-strong` |
| `--C-BORDER-FOCUS` | `border-border-focus`, `ring-border-focus` |
| `--C-STATUS-ERROR` | `text-status-error`, `bg-status-error` |
| `--C-STATUS-ERROR-BG` | `bg-status-error-bg` |
| (same for SUCCESS, WARNING, INFO) | |
| `--R-SIZE-1..6` | `p-r1`..`p-r6`, `m-r1`..`m-r6`, `gap-r1`..`gap-r6` (works with all spacing utilities) |
| `--H1..H6` | `text-h1`..`text-h6` |
| `--BodyText-1..3` | `text-body-1`, `text-body-2`, `text-body-3` |
| `--RADIUS-SM..XL` | `rounded-sm`, `rounded-md`, `rounded-lg`, `rounded-xl` |
| `--RADIUS-FULL` | `rounded-full` |
| `--SHADOW-SM..LG` | `shadow-sm`, `shadow-md`, `shadow-lg` |
| `--MOTION-DURATION-ENTER` | `duration-enter` |
| `--MOTION-EASE-ENTER` | `ease-enter` |
| `--DURATION-FAST` | `duration-fast` |

When extending utilities (adding new colors, etc.), also extend the `tailwindMergeExtension` config from `@batthewz/response-ui-react-components` so `cn()` collapses them correctly. See [the React package README](../packages/response-ui-react-components/README.md#adding-custom-tailwind-tokens).

---

## Authoring workflow

1. Copy the template:
   ```bash
   cp node_modules/@batthewz/response-ui-css/src/_theme-template.css ./src/themes/aurora.css
   ```
2. Change the selector to `:root[data-theme="aurora"]`.
3. Customize the required variables. Leave optionals commented out — uncomment only those you actually want to override.
4. Import after the main package CSS:
   ```css
   @import "@batthewz/response-ui-css";
   @import "./themes/aurora.css";
   ```
5. Register the theme name with `useTheme`:
   ```tsx
   const { setTheme } = useTheme({ themes: ["default", "aurora"] as const });
   setTheme("aurora");
   ```

Or generate from JSON:

```bash
bunx @batthewz/response-ui-css theme-from-json my-theme.json --name aurora > src/themes/aurora.css
```

Input shape:

```json
{
  "name": "aurora",
  "colorScheme": "dark",
  "vars": {
    "--C-CANVAS": "oklch(0.18 0.04 270)",
    "--C-PRIMARY": "oklch(0.6 0.15 220)",
    "...": "..."
  }
}
```
