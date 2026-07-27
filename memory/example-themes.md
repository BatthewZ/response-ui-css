# Example themes, and why the public entry stays thin

## The principle

This package defines exactly one theme: `default`, which *is* `:root`. Everything else —
including the worked examples that ship alongside — is a consumer theme.

That is not a naming preference. It is the whole claim of the theming feature. The moment a
shipped theme gets something a consumer's theme cannot have, the claim is false, and it becomes
false quietly: nothing fails, no test goes red, and the docs keep saying "re-skin from one page
of CSS" while the pre-installed themes enjoy a better deal.

## How it went wrong before, so you recognise the shape

Every step was locally reasonable and the sum was not:

- The public entry imported the example themes, so every consumer downloaded three palettes and
  nine font families they had not asked for. Convenient default; became the reason the examples
  could never be removed.
- Each example got its own subpath export, which made deleting one a breaking change.
- The docs called them "built-in", counted them ("all four themes"), and made that count the unit
  of QA — a retune was "finished" when a table covered four themes. A demo does not belong in an
  acceptance gate.
- The README put "define your own theme" *inside* the bullet list of built-ins, so the demos read
  as the feature and authoring read as a footnote.
- The sibling React package keyed chart ramps to example theme names, so a consumer's dark theme
  inherited a light chart ramp while two examples did not.

## The rules that follow

- **The public entries import no theme override layer.** `default` is `:root`; everything else is
  opt-in. Adding a theme `@import` to a public entry re-creates the problem.
- **Fonts follow their theme, but not inside the theme file.** The entry loads only what `default`
  names; a theme the consumer does not use must cost zero bytes. The obvious implementation —
  putting `@import url(...)` at the top of the theme file — **does not work**, and fails silently:
  CSS requires every `@import` to precede all other rules in the *flattened* stylesheet, and a
  theme is imported after the foundation, so the bundler drops it. Palette correct, typeface
  wrong, no error. Keep a theme's faces in a separate stylesheet the consumer imports first.
  This was caught only by building a real harness and grepping the output CSS — no unit test in
  either package can see it.
- **Anything under `examples/` is sample code and outside semver.** It may be changed or deleted
  without a breaking-change note. Do not let it acquire dependents.
- **Never enumerate the example names** in a selector, a type, a default, or prose implying a
  fixed set. If something must vary per theme, express it as a token the consumer also controls.
- **Contrast and palette work uses the examples as a regression corpus, not as a contract.** They
  are the only concrete themes available to measure, which is useful — but a green result across
  them says nothing about a consumer's theme, and any doc reporting those numbers must say so.

## The trap when measuring

A theme file that carries its own font `@import url(...)` will break a naive CSS parser: Google
Fonts URLs contain `;` inside the weight list, so stripping an at-rule "to the first semicolon"
leaves a fragment behind, and that fragment lands in the *next* rule's selector — silently
dropping every token in the file. A tool that reads theme files must strip `@import` to end of
line. The failure is silent and looks like "the numbers didn't change".
