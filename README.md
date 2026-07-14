# Rubik's Cube Solver

A single-file, static web app for solving a Rubik's Cube. Paint in the colors
of a scrambled cube (or hit Scramble for a random one), click **Solve Cube**,
then step through the solution move by move — manually or with auto-play.

Open `index.html` directly in a browser, or serve it with any static file
server. No build step, no dependencies.

## Features

- **Paint your cube**: click a color, then click stickers on the left cube to
  set up any valid scramble.
- **Scramble**: generates a random, solvable scramble for you.
- **Solve Cube**: runs a layer-by-layer solver (in a Web Worker, so the UI
  never freezes) and lists out the full move sequence.
- **Step through the solution**: click any move to jump to that point, or use
  **Play** to auto-animate through every step at your choice of speed
  (Slow/Normal/Fast). The right-hand cube always previews the result of the
  next move.
- **Reset All**: back to a solved cube.
- Drag either cube to rotate the view.

## Notation

Solutions use standard Singmaster notation, based on **Green = Front** and
**White = Up**:

| Move | Meaning |
|------|---------|
| `F`, `R`, `U`, `D`, `L`, `B` | Turn that face clockwise 90° |
| `'` (e.g. `F'`) | Counter-clockwise 90° ("prime") |
| `2` (e.g. `U2`) | 180° turn |

## How it works

Everything lives in `index.html`:

- The visible cube is a CSS 3D transform (`transform-style: preserve-3d`) —
  no canvas or WebGL.
- The solver runs inside a Web Worker built from an inline script (via
  `Blob` + `Worker`), so solving doesn't block the page.
- The solving algorithm is a layer-by-layer ("beginner's method") solver:
  cross → corners → F2L → OLL → PLL. It isn't move-count-optimized, just
  correct — expect roughly 100-150 moves per solution.

See `CLAUDE.md` for a deeper architectural walkthrough if you're working on
the code, and `HANDOVER.md` for the history/rationale behind the current
solver implementation.
