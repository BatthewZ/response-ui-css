# Retuning a colour token for contrast

## Move lightness only

Every contrast fix in this package so far has adjusted the OKLCH **L** channel and left **C** and
**H** byte-identical. That is what keeps a theme reading as its own orange / parchment / neon
rather than collapsing toward neutral. If a target cannot be met in lightness alone, that is a
finding to report, not a licence to reach for hue.

The direction is not uniform across themes. Light themes gain contrast by darkening the ink;
dark themes gain it by lightening. Expect the same token to move opposite ways in `events` and
`grimdark`.

## Which surfaces a token has to clear

The established floor is `SURFACE-0` through `SURFACE-2`. `SURFACE-3` is excluded on purpose:
the only places tokens meet it are inactive controls (the `disabled:bg-surface-3` recipe and its
CSS equivalents), which WCAG exempts. Extending a fix to `SURFACE-3` would drag every theme's
palette much darker for no accessibility gain.

## The accent trap

`--C-ACCENT` is load-bearing in two opposite roles at once: it is painted as ink directly on
surfaces, and it is the background behind `--C-TEXT-ON-ACCENT`. Separating it from the surface
moves it toward its own text. Always solve both constraints together, per theme, and check the
result against `--C-ACCENT-HOVER` as well — a selected-and-hovered control swaps the fill to
hover while the on-accent ink stays put, so on-accent must clear both fills.

On a dark theme there is a hard ceiling worth knowing: once the accent is light enough to read as
ink on `SURFACE-2`, even pure white on top of it falls short of 4.5:1. At that point the paired
`on-*` token has to invert to dark. Anchor it to a lightness the theme already uses for ink
rather than inventing one.

Whenever `--C-ACCENT` moves, `--C-ACCENT-HOVER` must move by the same delta or the hover state
inverts.

## Duplicated literals are the main hazard

Several tokens are byte-identical copies of others rather than `var()` references —
`--C-BORDER-FOCUS` repeats `--C-ACCENT` in most themes, `--C-TEXT-ON-PRIMARY` repeats
`--C-TEXT-PRIMARY` in `grimdark`, and one theme's `--C-ACCENT` also serves as its
`--C-STATUS-SUCCESS`. Grep the literal string, not just the token name, before and after any
move, and decide deliberately which copies travel together. The React package additionally
hard-codes some default-theme values into its own chart tokens, which will silently desync.

Theme files also embed colour literals inside `--SHADOW-*` and `--OVERLAY-*` definitions. Those
are decorative and normally stay put, but they are part of the same grep.

## Measure, don't estimate

Convert OKLCH through linear sRGB to relative luminance and compute the WCAG ratio directly.
Two details matter:

- Reproduce several already-known figures before trusting a new script.
- Darkening a saturated colour often pushes it outside sRGB at its declared chroma. Browsers
  gamut-map it, which changes the rendered luminance. Compute the ratio against the mapped
  colour, pessimistically across both plausible mapping strategies (constant-lightness chroma
  reduction, and per-channel clipping), so the result holds whichever a given engine uses. Leave
  the declared chroma alone — the rendered output is the same either way, and pre-reducing it
  would be a chroma edit.

Also measure composited washes (`color-mix` of an accent at 8–10% over a surface) separately;
they are their own pairing and they move when the accent moves.

## Report shape

A retune is only finished when there is a before/after ratio table covering **all four themes**,
every pairing being fixed, and every currently-passing pairing that involves a moved token.
Repairing four pairings while quietly breaking a fifth is a net loss.
