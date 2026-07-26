# Changelog

All notable changes to `@batthewz/response-ui-css` will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html). Until 1.0.0, breaking changes will bump the **minor** version.

## [Unreleased]

## [0.9.0] — 2026-07-25

### Fixed

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
