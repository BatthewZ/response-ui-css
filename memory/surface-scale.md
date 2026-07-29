# The surface ramp

## What it is

The surface tokens are a **nesting depth measured from the canvas, ascending**. The lowest
number sits closest to the page background; the highest is the most recessed region. That is
the whole invariant, and it is the only thing a theme author can be held to.

The *lightness* direction is deliberately not part of the contract. A light theme runs light →
dark across the ramp; a dark theme runs dark → light. Both satisfy it. So the lowest-numbered
surface is the lightest of the set in a light theme and the darkest in a dark one, and a
consumer's theme is free to do either.

## Why it must not be described as an elevation

Elevation is a fixed order — a dialog is always above a card, in every theme. The ramp is not,
because its lightness direction reverses. Assign the more-elevated component the lower number
and it renders lighter than its backdrop in one theme and darker in the other; the same
composition reads as a raised tile and as a hole punched in a panel depending on which theme is
active. Nobody notices, because everybody develops in one theme.

Elevation belongs to the shadow and border tokens, which point the same way regardless of the
palette. Say so wherever the ramp is documented — the metaphor travels further than the token
name, and a downstream page that inherits it will reason from it and reach conclusions no
measurement supports.

## Why a step is not a boundary

Adjacent rungs are a very weak cue — barely over 1:1 in every shipped theme, and the ramp's
full span is not much better. That is a property of the ramp, not a tuning mistake: a set of
backgrounds that separated strongly would stop reading as one page.

Two consequences worth stating in the contract rather than leaving to be rediscovered:

- **A component must never rely on a single step being visible.** Not for a hover wash, not for
  a selected state, not for a container's edge. Give it a border, a ring, or an ink change.
- **Two components on the same rung are supposed to look identical.** That is correct behaviour,
  not a collision — but it means a component that paints a rung and calls itself a bounded
  region is wrong unless it also draws an edge. This happens most often when a layout shell and
  a container primitive are both, correctly, on the rung named for containers.

## For a theme author

You may spread the rungs further apart than the shipped values. You may not reorder them: the
number is a distance from the canvas, and components choose a rung on that basis. If a theme
wants stronger separation between regions, the border token is the lever that actually reaches
it, and it reaches it in every theme rather than only the light ones.
