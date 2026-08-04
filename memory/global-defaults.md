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

Related: [contrast-retuning.md](./contrast-retuning.md) for which token pairs carry a
legibility guarantee and what it costs to move one.
