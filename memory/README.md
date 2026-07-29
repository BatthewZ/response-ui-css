# Memory index — `@batthewz/response-ui-css`

Durable guidance for agents working in this package. Principles only: no task lists, no file
paths or line numbers, nothing that goes stale when the code moves.

| Note | What it covers |
| --- | --- |
| [contrast-retuning.md](./contrast-retuning.md) | How to change a `--C-*` token for WCAG contrast: lightness-only moves, which surfaces a token must clear, the accent / on-accent conflict, duplicated colour literals, sRGB gamut mapping, and what a finished report looks like. |
| [surface-scale.md](./surface-scale.md) | What the surface ramp actually is (a nesting depth measured from the canvas, not an elevation), why its lightness direction reverses between light and dark themes, and why a single step can never be a component's boundary. Read before describing, reordering or retuning the surface tokens. |
| [example-themes.md](./example-themes.md) | Why `default` is the only theme this package defines, why the public entry imports no theme, and the shape by which example themes quietly become load-bearing. Read before adding anything theme-shaped to an entry file. |
