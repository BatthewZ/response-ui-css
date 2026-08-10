# Package-wide default rules

This package ships a handful of rules that paint something the browser would otherwise
paint for itself — scrollbars, the text-selection highlight, the ring-offset gap. They are
not components and they are not tokens; they are *defaults*, and they have their own three
failure modes.

## 1. An unlayered default outranks every Tailwind utility

Cascade layers beat specificity. A rule written outside any layer wins over anything in
`@layer utilities` no matter how specific the utility is, so an unlayered global default
cannot be overridden by the utility a consumer would naturally reach for — they are left
with `!important` or a more specific selector, for a rule that was only ever meant to be a
starting point.

Put a default that a utility should be able to beat in `@layer base`. Keep unlayered only
what a consumer is not expected to override piecemeal. This is a decision per rule, not a
house style: some of the globals here are unlayered on purpose.

## 2. Alias an existing token pair; do not mint new literals

A default that needs a colour should read from tokens the contract already defines rather
than declaring its own values. Aliasing means a theme that retunes the source carries the
default with it for free, and the default inherits whatever legibility guarantee the source
pair already carries instead of asking every theme author for another measurement. It also
keeps the token overridable at both levels — the alias, or the thing it points at.

The cost of the other choice is paid per theme, forever, and is invisible until someone
writes a theme and finds one corner of the UI still wearing the old palette.

## 3. A pseudo-element rule gets local scoping for free

Highlight pseudo-elements resolve custom properties against the element the pseudo
originates from. So a single rule with no selector beyond the pseudo-element serves both
the global default *and* every scoped variation: re-declare the token on a subtree and the
subtree re-tints. Adding selectors, or a per-scope utility, to achieve that is work the
cascade was already doing.

## 4. Put an alias's default at the READ, not in a `:root` declaration

Section 2 says a default should alias an existing token. *Where you write that alias* decides
whether it survives scoped theming, and the two forms look interchangeable.

`var()` inside a custom-property **declaration** is substituted at the element the declaration
applies to. Declared in the `:root` block, an alias resolves against `:root`'s value once, and
descendants inherit the resulting *colour* — not the indirection. Re-pointing the upstream token
further down the tree never re-runs that substitution, so the alias quietly stops tracking the
thing it aliases for any theme scoped to a subtree.

Write the alias as a fallback at the place that reads it — `var(--THE-TOKEN, var(--THE-SOURCE))`
— and it is re-evaluated wherever it is painted. Identical where nothing overrides, correct
where something does, and an explicit override still wins because a fallback only fires when the
token is unset.

Two things to accept when you do this. The token is then **not declared anywhere**, so
`getComputedStyle` reports it empty until someone sets it, and a consumer's own
`var(--THE-TOKEN)` written without a fallback resolves to nothing — so only convert a token
nothing is expected to read from script, and say so in the changelog either way. And the failure
this prevents is invisible to a package whose own themes all apply at `:root`: that arrangement
puts the theme and the alias on the same element, where the cascade resolves the upstream first
and everything behaves. Scoped theming is the only thing that exposes it.

Related: [contrast-retuning.md](./contrast-retuning.md) for which token pairs carry a
legibility guarantee and what it costs to move one.
