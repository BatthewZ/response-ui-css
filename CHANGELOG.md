# Changelog

All notable changes to `@batthewz/response-ui-css` will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html). Until 1.0.0, breaking changes will bump the **minor** version.

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
