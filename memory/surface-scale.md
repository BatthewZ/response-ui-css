# The surface ramp

## What it is

The surface tokens run **raised → recessed**, in the **same lightness direction in every
theme**: rung 0 is the lightest of the four in a light theme *and* in a dark one, rung 3 the
darkest in both. Raised regions catch the light in either mode, which is why the direction does
not flip.

The canvas is **not** an endpoint of that scale. It is the page floor and it belongs between
rungs 1 and 2, so rungs 0–1 sit above the page and rungs 2–3 sit below it. Two rules, and they
are the whole invariant: do not reorder the rungs, and do not let the canvas collide with one.
Spacing is the theme author's.

## The failure this replaced, because it will be proposed again

The ramp used to be defined as "nesting depth from the canvas, ascending", with the lightness
direction left to the theme. It was self-consistent and it was wrong in practice, for two
compounding reasons:

- **The canvas sat at an endpoint.** Both shipped light themes made the canvas *byte-identical*
  to rung 0, so anything painted on rung 0 had no boundary against the page whatsoever. The
  first fix attempted was to move the affected components down a rung — which treats the
  symptom, darkens the component, and leaves the collision in place for every other rung-0
  consumer.
- **The direction was per-theme.** Under it, a light theme ran light → dark and a dark theme ran
  dark → light. So light-mode surfaces all *sank* into the page while dark-mode surfaces
  *lifted* off it. Cards read as grey holes in light mode and as raised tiles in dark mode from
  the same token. Nobody catches this, because everybody develops in one theme.

If someone proposes returning the direction to the theme author, that second bullet is the
counter-example to hand them.

## Retuning it is a contrast change, always

Rungs 0–2 carry text; rung 3 is exempt (disabled fills and tracks only). So any move of a rung
is a WCAG change and goes through the contrast playbook, measured, not estimated.

The direction of the risk is not symmetric, and it is easy to get backwards:

- **Light themes** — a rung that moves *darker* loses contrast against dark ink. The muted text
  token is the binding constraint and it sits almost exactly on 4.5:1 against rung 2, so rung 2
  has essentially no room to darken without moving the ink too.
- **Dark themes** — a rung that moves *lighter* loses contrast against light ink. Rung 0 is the
  binding one here, because it is both the lightest rung and the one cards and dialogs paint,
  making it by far the most text-bearing surface in the system.

That second case is the trap. Reversing a dark theme's existing four values to satisfy the
direction rule promotes the old *most-recessed* value — a rung the old contrast floor
deliberately excluded — into the position that carries the most text. It measures as a straight
WCAG failure on secondary and muted ink. Dark themes need re-spacing, not reversing: anchor the
new rung 0 at a lightness the theme's ink was already validated against, and open the new room
*below* the canvas instead.

## Why a step is not a boundary

Adjacent rungs are a weak cue — roughly 1.02–1.15:1 in the shipped themes, and the
canvas-to-rung-0 lift that produces the "white card on a grey page" read is only about
1.05–1.16:1. That is a property of the ramp, not a tuning mistake: a set of backgrounds that
separated strongly would stop reading as one page.

Two consequences worth stating in the contract rather than leaving to be rediscovered:

- **A component must never rely on a single step being visible.** Not for a hover wash, not for
  a selected state, not for a container's edge. Give it a border, a ring, or an ink change.
- **Two components on the same rung are supposed to look identical.** Cards and dialogs share
  rung 0 by design. That is not a collision — but it does mean a component that paints a rung
  and calls itself a bounded region is wrong unless it also draws an edge.

## Watch the tokens that are not rungs but track them

A theme's default border token is often tuned to sit near one end of the old ramp. After a
retune, check that it is still visible against whichever rung the cards now paint — a border
that lands on its own fill's lightness silently removes every card's edge, and the ramp's own
weak steps mean nothing else is left to bound the component.

The Tailwind ring-offset default also points at a surface rung. It was correct for free while
the canvas and rung 0 were identical; once they separate, it is right for controls on a sheet
and wrong for controls sitting directly on the page. Pick the case that dominates and say which
one you picked.
