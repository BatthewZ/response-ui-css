# Changelog

All notable changes to `@batthewz/response-ui-css` will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html). Until 1.0.0, breaking changes will bump the **minor** version.

## [0.11.0] — 2026-07-28

`package.json` is at `0.11.0`; the published version is `0.10.1`. Everything below is what
0.11.0 will carry when it is cut, including the focus-ring fix that had been sitting under

### Breaking

- **The example themes are no longer imported by the public entries, and moved to `src/examples/themes/`.**
  `events`, `grimdark` and `tech` were never part of the design language — they are worked
  demonstrations of the theme contract. Shipping them inside `src/index.css` made that untrue in three
  measurable ways: every consumer downloaded three palettes they had not asked for, every consumer
  loaded **nine Google Font families** for themes they were not using, and a consumer's own theme was
  visibly a second-class citizen next to three that came pre-installed. `default` is, and always was,
  the only theme this package defines — it _is_ `:root`.

  **What breaks.** (1) `@import "@batthewz/response-ui-css"` no longer defines
  `[data-theme="events"|"grimdark"|"tech"]`; an app relying on one now renders unthemed until it opts
  in. (2) The subpaths `./themes/events`, `./themes/grimdark` and `./themes/tech` are **removed**,
  replaced by `./examples/themes/<name>`. (3) `./fonts` and the main entry now load only the two
  families the `default` theme names (Poppins, Libertinus Mono); each example theme's nine faces
  moved to a sibling `./examples/themes/<name>-fonts` you import separately — see the next entry
  for why they are not inside the theme files.

  **Migration** — one line per theme you actually use:

  ```css
  @import "@batthewz/response-ui-css/examples/themes/grimdark-fonts"; /* MUST be first */
  @import "@batthewz/response-ui-css";
  @import "@batthewz/response-ui-css/examples/themes/grimdark"; /* was implicit */
  ```

  The fonts line is not optional and its position is not stylistic — see the next entry.

  If you use `@batthewz/response-ui-react-components` charts or `MediaCard` with those themes, also
  add `@import "@batthewz/response-ui-react-components/examples/theme-tuning";` — their `--C-CHART-*`
  ramps moved to an opt-in stylesheet there for the same reason.

  Anything under `src/examples/` is **sample code and outside semver.** It may change or be replaced
  without a breaking-change note. That is the point of it.

  > **Reading older entries below:** they refer to `src/themes/<name>.css`, which is now
  > `src/examples/themes/<name>.css`. Their file links (including the "Revert:" instructions) are
  > left as written, because they record what was true at the time — translate the path.

- **Each example theme's fonts moved out of `src/fonts.css` into a sibling `<name>-fonts.css`, which
  must be imported first.** `fonts.css` previously carried all eleven families — two for `default` and
  nine for the three examples — and the main entry loaded the lot, so every consumer paid for nine
  faces belonging to themes they were not using. It now carries only the `default` theme's two.

  The obvious alternative — putting each theme's `@import url(...)` at the top of its own theme file —
  looks correct and silently does not work, so it is worth stating: CSS requires every `@import` to
  precede all other rules in the _flattened_ stylesheet, and a theme file is imported after the
  foundation, so bundlers drop it. The palette applies, the typeface never changes, and nothing errors.
  The same constraint applies to your own theme; put font imports at the top of your app's CSS entry.

  ```css
  @import "@batthewz/response-ui-css/examples/themes/grimdark-fonts"; /* first */
  @import "@batthewz/response-ui-css";
  @import "@batthewz/response-ui-css/examples/themes/grimdark";
  ```

### Changed

- **The theme docs lead with authoring, not with the examples.** The README's `## Themes` section
  previously contained only a `setAttribute("data-theme", "grimdark")` snippet, while "Define your own
  theme" sat mis-indented _inside_ the "Four built-in themes" bullet list — so the demos read as the
  feature and authoring read as a footnote. Reversed.
- **The partial-theme rule no longer contradicts itself.** `AGENTS.md` said "No partial themes — all
  required colour / font / `color-scheme` variables must be present", while `docs/theme-contract.md`
  and `src/_theme-template.css` both said "Nothing is strictly required". Both were half right and a
  reader got opposite answers depending on which file they opened. Now stated once: mechanically a
  `[data-theme]` block only _overrides_, so nothing is required — but the core set (`color-scheme`,
  canvas, surfaces, and the `--C-TEXT-*` ink that must contrast them) is **coupled**, and moving one
  without the others produces a provably broken theme.
- `docs/theme-contract.md` gained a font-import step in the authoring workflow, and a note that
  `default` is the only theme name the design system has an opinion about.

### Fixed

- **`--C-BORDER-FOCUS` now meets WCAG 1.4.11 in every shipped theme.** The `events` and `grimdark`
  focus rings were byte-identical copies of those themes' _pre-retune_ `--C-ACCENT` literals, so the
  0.9.0 accent retune left them behind: measured **2.72:1** and **2.96:1** on `--C-SURFACE-0`,
  falling to 2.52 and 2.55 on `--C-SURFACE-2`, against a 3:1 floor for a non-text indicator. Retuned
  in lightness only — hue and chroma byte-identical, as with every retune before it — to
  `oklch(0.6474 0.1867 47.6)` and `oklch(0.5549 0.1905 27.52)`. All four themes now clear the floor
  on `--C-SURFACE-0`/`-1`/`-2` with headroom: **3.39/3.29/3.15** (`events`), **3.66/3.43/3.15**
  (`grimdark`), 3.68/3.52/3.34 (default, unchanged), 14.84/14.56/13.70 (`tech`, unchanged).

  The default theme was checked and deliberately **not** touched: its focus ring differs from its
  accent in lightness, chroma _and_ hue, and it already passes. That is a design choice, not drift.

  Ring-against-accent-fill was checked for regression and is not a real adjacency: filled buttons use
  `ring-offset-2`, which paints a `--C-SURFACE-0` band between the fill and the ring, so the pairing
  that occurs on screen is ring-against-surface — the one that improved.

## [0.9.0] — 2026-07-25

> Amended 2026-07-26 with the `--C-TEXT-MUTED` retune, and again the same day with the
> `--C-ACCENT` / `--C-TEXT-ON-ACCENT` / `--C-BORDER-STRONG` / `--C-STATUS-SUCCESS` retune below.
> 0.9.0 was cut but **never published** — npm's latest is 0.8.0 — so these are amendments to an
> unreleased section, not a rewrite of shipped history. Folding them in rather than cutting
> 0.10.0 keeps `@batthewz/response-ui-react-components`'s `^0.9.0` dependency and the renderer's
> peer range valid without a second bump through the chain.

### Fixed

- **Four more token pairings now meet WCAG AA / 1.4.11.** Same method as the `--C-TEXT-MUTED` retune below: **only lightness moved, on every token** — hue and chroma are byte-identical to before, so each theme keeps its own orange / red / green / grey.

  **`--C-BORDER-STRONG` on the surfaces (WCAG 1.4.11, ≥ 3:1).** This is the boundary of every form control — `Input`, `Textarea`, `Select`, `NumberInput`, `SearchInput`, `TagInput`, `OTPInput`, `Checkbox`, `Combobox`, `ColorPicker`, `MultiSelect` — plus the inset ring that is the _entire_ "today" signal in `Calendar` and the hover ring on `Stepper`. It failed in all four themes, worse than 1.8:1 everywhere.

  | Theme    | Before                        | After                         | on `SURFACE-0` / `-1` / `-2`      |
  | -------- | ----------------------------- | ----------------------------- | --------------------------------- |
  | default  | `oklch(0.8717 0.0093 258.34)` | `oklch(0.6446 0.0093 258.34)` | 1.47→3.30 · 1.41→3.16 · 1.34→3.00 |
  | events   | `oklch(0.8687 0.0043 56.37)`  | `oklch(0.6427 0.0043 56.37)`  | 1.44→3.23 · 1.40→3.13 · 1.34→3.00 |
  | grimdark | `oklch(0.364 0.0192 76.79)`   | `oklch(0.5211 0.0192 76.79)`  | 1.79→3.49 · 1.67→3.26 · 1.54→3.00 |
  | tech     | `oklch(0.2922 0.029 284.46)`  | `oklch(0.4968 0.029 284.46)`  | 1.41→3.25 · 1.39→3.19 · 1.30→3.00 |

  The floor is `SURFACE-0` through `SURFACE-2`, matching the `--C-TEXT-MUTED` precedent. `SURFACE-3` is excluded for the same reason: `border-strong` only meets it through the `disabled:bg-surface-3` recipe, and WCAG 2.2 §1.4.11 exempts inactive components. Those pairs land at 2.61–2.73:1.

  **`--C-STATUS-SUCCESS` as text (≥ 4.5:1).** The `success` `Badge` / `Alert` / `Toast` ink, `FileUpload`'s success message, and `StatCard`'s upward trend line (via the React package's `--C-TREND-UP` alias) all sat at ~3.15:1. `grimdark` and `tech` already passed and are untouched; `default` and `events` share one value, retuned once and mirrored.

  | Theme    | Before                        | After                         | on `SURFACE-1` / `STATUS-SUCCESS-BG` |
  | -------- | ----------------------------- | ----------------------------- | ------------------------------------ |
  | default  | `oklch(0.6271 0.1699 149.21)` | `oklch(0.5307 0.1699 149.21)` | 3.15→4.58 · 3.15→4.57                |
  | events   | `oklch(0.6271 0.1699 149.21)` | `oklch(0.5307 0.1699 149.21)` | 3.10→4.50 · 3.15→4.57                |
  | grimdark | unchanged                     | unchanged                     | 7.87 · 6.70                          |
  | tech     | unchanged                     | unchanged                     | 14.56 · 13.39                        |

  **`--C-ACCENT` as ink, and `--C-TEXT-ON-ACCENT` on it (≥ 4.5:1 each).** Accent is painted as body text in `Swimlane`'s "view all", `FileUpload`'s "browse", `Button`'s `link` variant, `Breadcrumbs` hover, the selected `Tabs` label and `MultiSelect`'s check glyph — accent-as-ink, not accent-as-fill. It is _simultaneously_ the fill behind `--C-TEXT-ON-ACCENT` in `Calendar`'s selected day, `Pagination`'s current page and the `Tabs` pill. **These two pull in opposite directions**: separating accent from the surface moves it toward its own text.

  | Theme    | `--C-ACCENT` before → after                                 | ink on `SURFACE-0`/`-1`/`-2`          | `ON-ACCENT` before → after                                | on `ACCENT` / `ACCENT-HOVER` |
  | -------- | ----------------------------------------------------------- | ------------------------------------- | --------------------------------------------------------- | ---------------------------- |
  | default  | unchanged                                                   | 5.17 · 4.95 · 4.70                    | unchanged                                                 | 5.17 · 6.70                  |
  | events   | `oklch(0.7049 0.1867 47.6)` → `oklch(0.5575 0.1867 47.6)`   | 2.72→4.85 · 2.63→4.70 · 2.52→**4.50** | unchanged (`oklch(1 0 0)`)                                | 2.80→5.00 · 3.56→6.47        |
  | grimdark | `oklch(0.5054 0.1905 27.52)` → `oklch(0.6618 0.1905 27.52)` | 2.96→5.69 · 2.77→5.32 · 2.55→4.89     | `oklch(0.8285 0.0414 83.1)` → `oklch(0.1684 0.0414 83.1)` | 3.81→5.69 · 4.89→**4.50**    |
  | tech     | unchanged                                                   | 14.84 · 14.56 · 13.70                 | unchanged                                                 | 14.84 · 11.32                |

  In `events` the two requirements happen to agree — `--C-TEXT-ON-ACCENT` is white, so darkening the accent improves both. **In `grimdark` they cannot both be satisfied with a light `--C-TEXT-ON-ACCENT` at all.** Lifting the accent far enough to read as ink on `SURFACE-2` puts its luminance above 0.236; against _pure white_ that is only 3.66:1, so no light on-accent value exists. The token is therefore inverted: it keeps grimdark's parchment hue and chroma (`0.0414 / 83.1`) and drops to the theme's own dark-ink lightness, `0.1684` — the same lightness as `--C-TEXT-INVERSE` and `--C-SURFACE-0`. Dark parchment on a lit red, rather than light parchment on a dark red.

  `--C-ACCENT-HOVER` moved with the accent in both themes, by the identical lightness delta, so the hover step is unchanged in size and direction. It is not optional: `Calendar`'s selected-day hover swaps the fill to `--C-ACCENT-HOVER` while `--C-TEXT-ON-ACCENT` stays, so on-accent must clear the hover fill too, and leaving hover behind would have inverted the state (a hover _lighter_ than rest in `events`).

  **Gamut note.** At their new lightnesses `events`' accent/accent-hover and the shared success green sit slightly outside sRGB at their declared chroma, so a browser gamut-maps them to roughly 82% / 89% of the written chroma. The ratios above are computed against the _mapped_ colour, pessimistically across both plausible mapping strategies (constant-lightness chroma reduction and per-channel clipping), so they hold either way. The declared chroma is left untouched rather than pre-reduced: the rendered result is identical, and writing a reduced chroma would be a chroma edit.

  **Not taken here, deliberately.** `--C-BORDER-FOCUS` is a byte-identical copy of the old `--C-ACCENT` in `events` and `grimdark` (and in `_theme-template.css`), so the focus ring no longer matches the accent in those two themes. It is a separate token with its own contract and was not in scope; re-syncing it is a design call. Likewise: accent-as-ink and success-as-ink still fail on `SURFACE-3` (4.10–4.26:1 and ~4.33:1), and the React package's `--C-CHART-1` / `--C-CHART-2` hard-code the old default accent and success values and will not track this change.

  **This changes rendered output for every consumer** of `bg-accent` / `text-accent` / `--C-ACCENT`, `--C-TEXT-ON-ACCENT`, `border-border-strong` / `--C-BORDER-STRONG`, and `text-status-success` / `bg-status-success` / `--C-STATUS-SUCCESS`, on `default`, `events` and `grimdark` (`tech` changes only its border). **Revert:** restore the "Before" values above in [src/tokens/colors.css](./src/tokens/colors.css), [src/themes/events.css](./src/themes/events.css), [src/themes/grimdark.css](./src/themes/grimdark.css) and [src/themes/tech.css](./src/themes/tech.css).

- **`--C-TEXT-MUTED` now meets WCAG AA on every surface it is actually painted on, in all four themes.** It previously failed AA _and_ AA-large everywhere — the best any theme managed was **2.59:1** against `--C-SURFACE-0`, and the worst was 2.06:1. Muted text is not decorative here: it is the placeholder in every text control, the outside-month day in `Calendar`, the tag-remove affordance in `MultiSelect`, and the secondary line in `FileUpload`, `StatCard`, `Breadcrumbs` and `Timeline`.

  | Theme    | Before                        | After                         | on `SURFACE-0` / `-1` / `-2` |
  | -------- | ----------------------------- | ----------------------------- | ---------------------------- |
  | default  | `oklch(0.7137 0.0192 261.32)` | `oklch(0.5451 0.0192 261.32)` | 4.95 · 4.74 · 4.50           |
  | events   | `oklch(0.7161 0.0091 56.26)`  | `oklch(0.5435 0.0091 56.26)`  | 4.85 · 4.70 · 4.50           |
  | tech     | `oklch(0.3957 0.0369 284.59)` | `oklch(0.5937 0.0369 284.59)` | 4.87 · 4.78 · 4.50           |
  | grimdark | `oklch(0.4517 0.0252 85.94)`  | `oklch(0.6186 0.0252 85.94)`  | 5.23 · 4.90 · 4.50           |

  **Only lightness moved** — every theme keeps its own chroma and hue, so the muted tone still reads as that theme's grey/parchment/blue rather than a neutral. Note the direction is _not_ uniform: the light themes darken, and `tech` and `grimdark` **lighten**, because on a dark surface contrast is gained by moving up.

  The floor is `SURFACE-0` through `SURFACE-2`. `SURFACE-3` is excluded deliberately: every place muted text meets it is an inactive control (`.combobox-input:disabled`, `.multiselect[data-disabled]`, and the `disabled:bg-surface-3` recipe shared by `Input`/`Textarea`/`Select`), which WCAG 2.2 §1.4.3 exempts. Those pairs land at 3.92–4.10:1.

  **Known residual on the dark themes.** `tech` and `grimdark` cannot fit three AA-passing text steps in the lightness they have — their own `--C-TEXT-SECONDARY` only reaches 5.76:1 and 5.95:1. Muted therefore ends up close to secondary (0.041 and 0.033 apart in OKLCH L) and the two are now hard to tell apart, though the ordering is still correct. Widening that gap means lifting `--C-TEXT-SECONDARY` on both themes, which is a separate decision and is not taken here.

  **This changes rendered output for every consumer**, on every theme, anywhere `text-fg-muted` / `--C-TEXT-MUTED` is used. **Revert:** restore the four values in the "Before" column above ([src/tokens/colors.css](./src/tokens/colors.css), [src/themes/events.css](./src/themes/events.css), [src/themes/tech.css](./src/themes/tech.css), [src/themes/grimdark.css](./src/themes/grimdark.css)).

- **The focus-ring offset no longer paints white on every dark theme.** Tailwind registers `--tw-ring-offset-color` as an `@property` with an initial value of `#fff` and `inherits: false`, so every `ring-offset-*` utility punched an un-themeable white gap between an element and its focus ring — correct on a white page, a bright halo on `grimdark` and `tech`. `src/base.css` now defaults the variable to `var(--C-SURFACE-0)` for `*`, `::before`, `::after` and `::backdrop`, so the gap tracks the theme. Measured in a real engine before and after: a focused control reported `--tw-ring-offset-color: oklch(1 0 0)` in every theme beforehand, and `oklch(0.1684 0 0)` — grimdark's `--C-SURFACE-0` — afterwards.

  The rule is deliberately inside `@layer base`: an unlayered rule would out-rank Tailwind's `utilities` layer and make an explicit `ring-offset-<color>` utility unable to override it. Verified by walking the CSSOM for the rule and asserting its enclosing layer.

  **This changes rendered output for any consumer using a `ring-offset-*` utility.** If you were relying on the white gap, restore it per call site with `ring-offset-white`, or globally by re-declaring `--tw-ring-offset-color` after this package's import. **Revert:** delete the `@layer base` block under "Focus ring offset" in [src/base.css](./src/base.css).

## [0.7.0] and [0.8.0] — not documented

Both versions were published to npm and neither was recorded here, so this file jumps from 0.6.0 to 0.9.0. Nothing has been reconstructed for them after the fact: no record of their contents was kept at release time, and inventing one would be worse than the gap. The published tarballs and this repository's commit history between the 0.6.0 and 0.8.0 releases are the only account of what changed.

## [0.6.0] — 2026-06-13

Tightens the universal-contract boundary: domain tokens (data-viz + single-component feel) move out to the libraries that own them, while the genuinely universal aspect ratios stay — renamed to shed the domain prefix. Themes may only tune universal tokens, so the matching chart/media-card overrides are gone too. Pair with `@batthewz/response-ui-react-components`, which re-homes the relocated tokens.

### Removed

- **Trend tokens** — `--C-TREND-UP` / `--C-TREND-UP-BG` / `--C-TREND-DOWN` / `--C-TREND-DOWN-BG` and their `@theme inline` mappings. Pure data-viz aliases of status; they relocate to `@batthewz/response-ui-react-components`.
- **Categorical chart palette** — `--C-CHART-1`..`--C-CHART-5` and their `@theme inline` mappings, plus the dark-tuned per-theme overrides in [src/themes/grimdark.css](./src/themes/grimdark.css) and [src/themes/tech.css](./src/themes/tech.css). Data-viz palette; relocates to the React package.
- **Media-card feel + carousel config** — `--MEDIA-CARD-HOVER-SCALE` / `--MEDIA-CARD-HOVER-LIFT` (and their per-theme overrides in [grimdark.css](./src/themes/grimdark.css), [tech.css](./src/themes/tech.css), [events.css](./src/themes/events.css)) and `--MEDIA-CAROUSEL-PEEK` / `--MEDIA-CAROUSEL-GAP`. Single-component config; relocates to the React package.
- **Trend / chart docs + template entries** — dropped from [docs/theme-contract.md](./docs/theme-contract.md) and [src/\_theme-template.css](./src/_theme-template.css); they re-document under the React package.

### Changed

- **Renamed the universal aspect ratios** — `--MEDIA-ASPECT-WIDE` → `--ASPECT-WIDE` and `--MEDIA-ASPECT-SQUARE` → `--ASPECT-SQUARE`, dropping the domain-smuggling `MEDIA-` prefix. The `@theme inline` mappings change correspondingly, so the utilities are now `aspect-wide` / `aspect-square`. The two ratios moved from `src/tokens/media.css` to the new [src/tokens/aspect.css](./src/tokens/aspect.css) (`src/tokens/media.css` is removed; [src/tokens/index.css](./src/tokens/index.css) now imports `aspect.css`).

  **Migration:** swap `--MEDIA-ASPECT-WIDE` → `--ASPECT-WIDE`, `--MEDIA-ASPECT-SQUARE` → `--ASPECT-SQUARE`, and utility classes `aspect-media-aspect-wide`/etc. → `aspect-wide` / `aspect-square`.

- **Dropped `--MEDIA-ASPECT-POSTER`** (`2 / 3`) — a media-domain ratio rather than a universal layout primitive. It moves to `@batthewz/response-ui-react-components`.

## [0.5.0] — 2026-06-11

### Removed

- **Sideways `@source` directives for `@batthewz/response-ui-react-components`** from [src/index.css](./src/index.css) and [src/index-no-fonts.css](./src/index-no-fonts.css). The `../../response-ui-react-components/…` paths assumed npm's hoisted layout and silently matched nothing under isolated stores (bun, pnpm). Tailwind source registration now lives inside the components package itself — its `styles` entry carries a self-relative `@source` that resolves under any layout.

  **Migration:** pair with `@batthewz/response-ui-react-components` ≥ 0.3.0 (first release with the self-relative `@source`). If you must stay on an older components release, add a manual `@source "<path-to>/node_modules/@batthewz/response-ui-react-components/src/**/*.{ts,tsx}";` to your app CSS.

## [0.4.0]

### Breaking

- **Removed the `theme-from-json` CLI** (`scripts/theme-from-json.mjs`) and its `bin` entry. The script was a thin string-builder that emitted a `:root[data-theme="…"]` block from a JSON token map — with no validation against the theme contract. It was originally a target for a `ThemeEditor` UI that hasn't existed in this package since extraction (see 0.2.0 entry below), and no sibling project under `@batthewz/` consumed it.

  **Migration:** copy [src/\_theme-template.css](./src/_theme-template.css) and edit it directly, or hand-author a `:root[data-theme="…"]` block per [docs/theme-contract.md](./docs/theme-contract.md). Both paths are strictly better than the CLI — the template lists every required and optional variable with comments; the CLI didn't.

### Removed

- `scripts/theme-from-json.mjs` and the `scripts/` directory.
- `bin.theme-from-json` and the `prepublishOnly` syntax-check from [package.json](./package.json); `scripts` removed from the `files` array.
- CLI references from [README.md](./README.md) ("Generate from JSON" on-ramp) and [docs/theme-contract.md](./docs/theme-contract.md) ("Or generate from JSON" + JSON input shape).
- CLI references from [AGENTS.md](./AGENTS.md) (the `Where things live` table row and the `theme-from-json CLI` section).

## [0.2.0] — 2026-06-05

### Breaking

- **Per-component CSS has been moved out of this package.** The 24 component CSS files (Accordion, AppShell, Breadcrumbs, Button, Carousel, DropdownMenu, EmptyState, FileUpload, Hero, MasonryGrid, MediaCard, Pagination, Popover, ProgressBar, SearchInput, Skeleton, Spotlight, StatCard, Swimlane, Table, Tabs, ThemeSwitcher, Timeline, Tooltip) now live in [`@batthewz/response-ui-react-components`](../response-ui-react-components/), co-located with each `.tsx`. The `src/components/` directory and its `index.css` aggregator are gone.

  This package is now **strictly the design-system foundation**: tokens, themes, responsive scales, animations, base styles, and fonts. The framework-agnostic story is now honest — `@import "@batthewz/response-ui-css";` gives you primitives you can build any UI on top of, from any framework.

  **Impact:**
  - **React users:** add `@import "@batthewz/response-ui-react-components/styles";` after your existing `@import "@batthewz/response-ui-css";`. See that package's [CHANGELOG](../response-ui-react-components/CHANGELOG.md) for the migration guide.
  - **Non-React users** (Astro, Phoenix, Rails, plain HTML): no action required. You weren't using the per-component classes anyway — you were here for tokens, themes, and responsive scales, all of which are unchanged.

### Removed

- `src/components/` directory and its `index.css` aggregator (now in [`@batthewz/response-ui-react-components`](../response-ui-react-components/)).
- Three unused / showcase-only CSS files:
  - `demo-toc.css` (202 lines) — no React counterpart; leftover from when these packages were extracted from a parent project.
  - `theme-bubble.css` (113 lines) — no React counterpart; same.
  - `link.css` (8 lines) — defined a `.link` class that nothing referenced. `Button.tsx`'s `link` variant is implemented inline with Tailwind utilities (`text-accent hover:underline font-bold`).
- Stale "ThemeEditor in the showcase" references from [README.md](./README.md) and [docs/theme-contract.md](./docs/theme-contract.md) — no `ThemeEditor` or showcase component has existed in this package since extraction.

### Changed

- **README pitch reframed** from "design tokens, themes, and component styles" to "the foundation layer — tokens, themes, responsive scales, and animations" — accurately describing what now ships.
- **"What ships" list updated** — removed the "Component styles" bullet, added a "Base styles" bullet.
- **AGENTS.md internal pipeline list** re-numbered (was 9 items including component CSS; now 8). Added a rule clarifying that per-component CSS belongs in the React package, not here.
- **[src/index.css](./src/index.css) and [src/index-no-fonts.css](./src/index-no-fonts.css) header comments** updated to point at the React package for per-component CSS and to remove the `@import "./components/index.css"` line.

### Added

- **Live demo link** in the README pointing at <https://ai-website-starter.benmatthews-it.workers.dev/demo>.
- **The responsive scale** documentation section in the README, explaining:
  - `r` = "Responsive" — one token, automatic step-up at 40rem
  - The `1` = most-significant numbering convention shared across spacing, headings, and body text
  - Headings (`text-h1`..`text-h6`) mirror HTML `<h1>`–`<h6>` semantics
  - 6 heading sizes + 3 body sizes + 6 spacing values
  - Line-heights paired with every size and re-tuned per breakpoint
  - `font-bold` / `font-semibold` auto-adjust at the desktop breakpoint for legibility
- **Expanded responsive-scale sections** in [docs/theme-contract.md](./docs/theme-contract.md) covering the same conventions in token-level detail, with concrete numeric examples and override guidance.

## [0.1.0] — Initial release

Initial public release.
