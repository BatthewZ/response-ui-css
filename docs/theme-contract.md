# Theme contract

A theme is a CSS rule that overrides design-system tokens under a `data-theme` selector. The contract below is the authoritative list of tokens a theme can set. Nothing is strictly required — the package ships a complete default theme on `:root`, so a `data-theme` block only _overrides_, and any token you omit inherits its default. The **core** tokens (color set + fonts) are semantically coupled, though: set them together, or you'll get an incoherent result (e.g. a dark `color-scheme` with a light-theme text color). The rest are independent and rarely need changing.

> Selector convention: `:root[data-theme="<name>"]`. The `default` theme IS `:root` itself (no override layer); switching to `default` removes the `data-theme` attribute.

> `default` is the only theme name the design system has an opinion about. The
> `events`, `grimdark` and `tech` files under `src/examples/themes/` are worked
> examples of this contract — no entry imports them, no code depends on them, and
> they sit outside semver. Your theme and theirs are the same kind of object.

---

## Core (semantic tokens)

These have `:root` defaults like everything else, but they're the coupled set you'll usually define together for a coherent theme.

### Color scheme

```css
color-scheme: light | dark;
```

Drives form-control colors, default scrollbar appearance, etc. Set this — don't omit it.

### Brand colors

| Variable              | Notes                                       |
| --------------------- | ------------------------------------------- |
| `--C-CANVAS`          | Page background                             |
| `--C-PRIMARY`         | Brand primary fill                          |
| `--C-PRIMARY-HOVER`   | Primary hover state                         |
| `--C-PRIMARY-ACTIVE`  | Primary pressed state                       |
| `--C-SECONDARY`       | Secondary fill                              |
| `--C-SECONDARY-HOVER` | Secondary hover state                       |
| `--C-ACCENT`          | Brand accent (e.g. links, focus indicators) |
| `--C-ACCENT-HOVER`    | Accent hover state                          |

All colors are OKLCH. Use OKLCH in your custom theme.

### Surfaces (layered backgrounds)

The scale runs **raised → recessed**, and it runs the **same lightness direction in every theme**: `--C-SURFACE-0` is the lightest of the four in a light theme _and_ in a dark one, `--C-SURFACE-3` the darkest in both. Raised regions catch the light either way — that is why the direction does not flip.

`--C-CANVAS` is not the end of the scale. It is the page floor, and it belongs **between rungs 1 and 2**:

```
  SURFACE-0   ┐ raised above the page
  SURFACE-1   ┘
  ·············  --C-CANVAS  ·············
  SURFACE-2   ┐ recessed into the page
  SURFACE-3   ┘
```

| Variable        | Use                                                                              |
| --------------- | -------------------------------------------------------------------------------- |
| `--C-SURFACE-0` | The raised sheet — cards, dialogs, popovers, menus, sidebars, resting input fills |
| `--C-SURFACE-1` | Still raised, one step less. Panels nested inside a sheet; table header rows      |
| `--C-SURFACE-2` | Mildly recessed. Hover washes, chips, badges, wells nested inside a sheet         |
| `--C-SURFACE-3` | The deepest wells — progress and slider tracks, disabled fills                    |

That is the whole invariant: **do not reorder the rungs, and do not let the canvas collide with one.** Spacing is yours — you may spread the rungs further apart than the shipped values.

**A rung is still not an elevation.** A dialog is not "above" a card because both are on rung 0 — they are supposed to look alike. Elevation is carried by `--SHADOW-*` and `--C-BORDER-DEFAULT`; separate two things on the same rung with one of those, never by borrowing a neighbouring rung.

Plan for adjacent rungs being a **weak** cue: one step measures 1.02–1.15:1 across the four measured themes, and the canvas-to-rung-0 lift — the "white card on a grey page" step — is 1.05–1.16:1. Nothing should depend on a single step being visible on its own.

**Do not pin the canvas at pure white or pure black.** It reads as a tempting extreme, but it leaves the recessed rungs nowhere to go, and any component that paints rung 0 then has no boundary against the page at all. Both shipped dark examples used to sit at `oklch(0 0 0)` and were retuned for exactly this reason.

### Text

| Variable              | Use                                                      |
| --------------------- | -------------------------------------------------------- |
| `--C-TEXT-PRIMARY`    | Default body text                                        |
| `--C-TEXT-SECONDARY`  | De-emphasized text (captions, helpers)                   |
| `--C-TEXT-MUTED`      | Most-muted (placeholders, hints)                         |
| `--C-TEXT-INVERSE`    | Text on a dark surface in a light theme (and vice versa) |
| `--C-TEXT-ON-PRIMARY` | Text drawn on `--C-PRIMARY` fill                         |
| `--C-TEXT-ON-ACCENT`  | Text drawn on `--C-ACCENT` fill                          |

### Borders

| Variable             | Use                            |
| -------------------- | ------------------------------ |
| `--C-BORDER-DEFAULT` | Default border (cards, inputs) |
| `--C-BORDER-STRONG`  | Higher-contrast border         |
| `--C-BORDER-FOCUS`   | Focus ring color               |

### Status

Each status has a foreground color and a tinted background:

| Foreground           | Background              |
| -------------------- | ----------------------- |
| `--C-STATUS-ERROR`   | `--C-STATUS-ERROR-BG`   |
| `--C-STATUS-SUCCESS` | `--C-STATUS-SUCCESS-BG` |
| `--C-STATUS-WARNING` | `--C-STATUS-WARNING-BG` |
| `--C-STATUS-INFO`    | `--C-STATUS-INFO-BG`    |

### Text selection

| Variable                | Use                                        |
| ----------------------- | ------------------------------------------ |
| `--C-SELECTION`         | The highlight painted behind selected text |
| `--C-TEXT-ON-SELECTION` | Selected text drawn on that highlight      |

Both default to the accent pair — `var(--C-ACCENT)` and `var(--C-TEXT-ON-ACCENT)` — so a theme that only retunes the accent moves the highlight with it, and the pairing below covers it for free. Set them when you want the highlight to differ from the accent; move both when you do.

The package applies them in a single global `::selection` rule. There is nothing to add for a **local** highlight either: `::selection` resolves custom properties against the element the selection originates from, so re-declaring `--C-SELECTION` on a subtree re-tints selection inside it.

```css
.brochure {
  --C-SELECTION: oklch(0.85 0.16 95);
  --C-TEXT-ON-SELECTION: oklch(0.2 0 0);
}
```

### The contrast pairing

A few of the colour tokens above are meant to be used **together**: a fill with the text that sits on it, and each status colour with its tinted background.

| Draw this…                  | …on this                |
| --------------------------- | ----------------------- |
| `--C-TEXT-ON-PRIMARY`       | `--C-PRIMARY`           |
| `--C-TEXT-ON-ACCENT`        | `--C-ACCENT`            |
| `--C-TEXT-ON-SELECTION`     | `--C-SELECTION`         |
| `--C-STATUS-*` (foreground) | `--C-STATUS-*-BG`       |

**The intention.** Each pair is designed to read against _itself_ — the `on-*` text is chosen to be legible on its own fill, and consuming components use these pairings for their defaults. That is the whole of it. It is a **convention, not a measured ratio**: this file names the pairings, never a number, and says nothing about a fill set against a `surface-*`, against `--C-CANVAS`, or over an image. (It is also why a focus ring earns its keep — a fill is only reliably distinct from its _paired text_, not from whatever surface it happens to sit on.)

**You can do whatever you like.** Re-tint any token; the pairing is a default, not a rule the system enforces. Two things to keep in mind if you stray from it: when you redefine a pair, move both so they still read against each other; and when you put a fill token somewhere other than under its paired text — on a surface, over a photo — the pairing says nothing about that, so check the contrast yourself.

### Typography

| Variable                   | Notes                                          |
| -------------------------- | ---------------------------------------------- |
| `--DEFAULT-FONT`           | Body font-family                               |
| `--DEFAULT-MONO-FONT`      | Monospace font-family                          |
| `--HEADING-FONT`           | Heading font-family (often = `--DEFAULT-FONT`) |
| `--HEADING-LETTER-SPACING` | `normal` or a `<length>` like `0.06em`         |
| `--HEADING-TEXT-TRANSFORM` | `none` / `uppercase` / `lowercase`             |

If you use a font that's not already loaded by `@batthewz/response-ui-css`, import the font-face yourself before your theme CSS.

---

## Optional

Override only what you want to change. Omitted tokens keep their package defaults — static ones from `:root`, responsive ones across both the base and the `@media (width >= 40rem)` step.

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

| Family    | Tokens                                                                              | Count | Notes                                                                                                                                                        |
| --------- | ----------------------------------------------------------------------------------- | ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Headings  | `--H1, --H1-line-height` … `--H6, --H6-line-height`                                 | 6     | Numbering mirrors HTML `<h1>`–`<h6>`, so `--H2` is what's applied to `<h2>` (and to `.h2` / `text-h2`).                                                      |
| Body text | `--BodyText-1, --BodyText-1-line-height` … `--BodyText-3, --BodyText-3-line-height` | 3     | Large / base / fine. Drives `text-body-1`..`text-body-3`.                                                                                                    |
| Spacing   | `--R-SIZE-1` … `--R-SIZE-6`                                                         | 6     | `R-SIZE-1` is the largest gap, `R-SIZE-6` the tightest. Drives `p-r1`..`p-r6`, `m-r1`..`m-r6`, `gap-r1`..`gap-r6`, and every other Tailwind spacing utility. |

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

### Aspect ratios

```
--ASPECT-WIDE, --ASPECT-SQUARE
```

Generic layout primitives for any image/video. Map to `aspect-wide` (`16 / 9`) and `aspect-square` (`1 / 1`).

---

## Tailwind utility mapping

`@theme inline` blocks expose tokens to Tailwind utilities. The mapping follows a `--SCREAMING-CASE-FOR-TOKEN` → `lowercase-kebab-for-utility` convention with one wrinkle: `--C-TEXT-*` → `text-fg-*`.

| Token                                   | Utility                                                                                  |
| --------------------------------------- | ---------------------------------------------------------------------------------------- |
| `--C-CANVAS`                            | `bg-canvas`                                                                              |
| `--C-PRIMARY` (and `-HOVER`, `-ACTIVE`) | `bg-primary`, `bg-primary-hover`, `bg-primary-active` (also `text-`, `border-`, `ring-`) |
| `--C-SECONDARY` (and `-HOVER`)          | `bg-secondary`, `bg-secondary-hover`                                                     |
| `--C-ACCENT` (and `-HOVER`)             | `bg-accent`, `bg-accent-hover`                                                           |
| `--C-SURFACE-0..3`                      | `bg-surface-0`, `bg-surface-1`, `bg-surface-2`, `bg-surface-3`                           |
| `--C-TEXT-PRIMARY`                      | `text-fg-primary`                                                                        |
| `--C-TEXT-SECONDARY`                    | `text-fg-secondary`                                                                      |
| `--C-TEXT-MUTED`                        | `text-fg-muted`                                                                          |
| `--C-TEXT-INVERSE`                      | `text-fg-inverse`                                                                        |
| `--C-TEXT-ON-PRIMARY`                   | `text-fg-on-primary`                                                                     |
| `--C-TEXT-ON-ACCENT`                    | `text-fg-on-accent`                                                                      |
| `--C-BORDER-DEFAULT`                    | `border-border-default`, `ring-border-default`                                           |
| `--C-BORDER-STRONG`                     | `border-border-strong`                                                                   |
| `--C-BORDER-FOCUS`                      | `border-border-focus`, `ring-border-focus`                                               |
| `--C-STATUS-ERROR`                      | `text-status-error`, `bg-status-error`                                                   |
| `--C-STATUS-ERROR-BG`                   | `bg-status-error-bg`                                                                     |
| (same for SUCCESS, WARNING, INFO)       |                                                                                          |
| `--ASPECT-WIDE`                         | `aspect-wide`                                                                            |
| `--ASPECT-SQUARE`                       | `aspect-square`                                                                          |
| `--R-SIZE-1..6`                         | `p-r1`..`p-r6`, `m-r1`..`m-r6`, `gap-r1`..`gap-r6` (works with all spacing utilities)    |
| `--H1..H6`                              | `text-h1`..`text-h6`                                                                     |
| `--BodyText-1..3`                       | `text-body-1`, `text-body-2`, `text-body-3`                                              |
| `--RADIUS-SM..XL`                       | `rounded-sm`, `rounded-md`, `rounded-lg`, `rounded-xl`                                   |
| `--RADIUS-FULL`                         | `rounded-full`                                                                           |
| `--SHADOW-SM..LG`                       | `shadow-sm`, `shadow-md`, `shadow-lg`                                                    |
| `--MOTION-DURATION-ENTER`               | `duration-enter`                                                                         |
| `--MOTION-EASE-ENTER`                   | `ease-enter`                                                                             |
| `--DURATION-FAST`                       | `duration-fast`                                                                          |

When extending utilities (adding new colors, etc.), expose them via `@theme inline` so the classes are generated — see [docs/extending.md](./extending.md). If your project uses a `tailwind-merge`-based class helper, also register the new utility names there so conflicting classes still collapse correctly.

---

## Authoring workflow

1. Copy the template:
   ```bash
   cp node_modules/@batthewz/response-ui-css/src/_theme-template.css ./src/themes/aurora.css
   ```
2. Change the selector to `:root[data-theme="aurora"]`.
3. Customize the core tokens. Leave the rest commented out — uncomment only those you actually want to override.
4. Import the font faces your theme names — at the **top of your app's CSS entry**, above
   the foundation import. The main entry loads only the two families the `default` theme
   uses, so anything else is yours to bring:
   ```css
   @import url("https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@300..700&display=swap");
   ```
   **Not inside the theme file.** CSS requires every `@import` to precede all other rules in
   the flattened stylesheet, and your theme file is imported *after* the foundation — so an
   `@import` there lands mid-bundle and is dropped without error. The symptom is a correct
   palette in the wrong typeface. Each example theme keeps its fonts in a separate
   `<name>-fonts.css` for exactly this reason.
5. Import after the main package CSS:
   ```css
   @import "@batthewz/response-ui-css";
   @import "./themes/aurora.css";
   ```
6. Activate it by setting `data-theme` on `<html>`:
   ```html
   <html data-theme="aurora">
     <!-- etc -->
   </html>
   ```
   or dynamically in `js`:
   ```ts
   document.documentElement.setAttribute("data-theme", "aurora");
   // removeAttribute("data-theme") to return to the default theme
   ```
