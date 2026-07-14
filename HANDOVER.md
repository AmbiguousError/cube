# Handover: Rubik's Cube solver fix

## What was broken

`index.html` is a single-file Rubik's Cube UI (paint stickers, scramble,
solve, step through moves). Two independent things were broken:

1. **`R()` and `B()` move functions** (used for scrambling and the move
   preview) had a transposition bug — a copy/paste error swapped two
   assignments, so turning either face four times didn't return to the
   solved state, which is physically impossible for a real cube move.
2. **The solving algorithm** (`get_solve_string` in the Web Worker) was an
   unbounded, essentially unpruned recursive search (depth starting at 99,
   12-way branching per level) that blew the JS heap instead of ever
   producing a solution.

## What was done

- Fixed the two swapped assignments in `R()`/`B()`. Verified against an
  independent 3D-rotation-matrix derivation of all 6 face turns (not just
  "it looks right" — actually re-derived the correct permutation from
  geometry and diffed against the existing code).
- Replaced the entire solving algorithm with a layer-by-layer ("beginner's
  method") solver: D-cross → D-corners → F2L edges → OLL (orient edges,
  then corners) → PLL (permute corners, then edges).
- **Every algorithm used was empirically verified** by BFS search against
  the actual move engine rather than trusted from memory — several
  "textbook" algorithms recalled from training data turned out subtly
  wrong when checked this way (twisting corners they should only permute,
  disturbing already-placed pieces from an earlier stage, etc.). See
  "Lessons" below if extending this further.
- Tested with 3000+ random scrambles (lengths 1–100 moves) at the
  algorithm level (100% solve rate), and end-to-end in a real headless
  browser via Playwright: scramble → solve → click through every step →
  cube verified genuinely solved via DOM inspection (not just the success
  message), plus manually painting a scrambled layout via the actual
  color-palette + sticker clicks → solve also confirmed working.

## Lessons (useful if this solver needs further changes)

- **Never test a move sequence by overwriting cells with real face letters
  (`'D'`, `'R'`, etc.) on an otherwise-solved cube.** The solved cube's own
  default labels are those same letters, so a check like "does 'D' end up
  at D5" can pass by coincidence, picking up an unrelated cell's default
  value. Always use unique lowercase dummy tags (`'d'`, `'x'`, `'y'`) that
  can't collide with the uppercase `U/D/F/B/L/R` defaults.
- **"Correctly places the target piece" isn't enough to verify.** An
  algorithm can look right while silently disturbing (a) other
  already-placed pieces of the *same* stage, or (b) the *orientation* of
  pieces whose orientation an *earlier* stage already fixed, even while
  landing them in the technically-right slot. Always check the full
  environment (D-layer, F2L, previously-fixed pieces) stays intact, not
  just the one piece being tested.
- **Greedy "apply algorithm, nudge by one U turn if not done, repeat"
  loops can cycle forever** even when a solution exists — some OLL/PLL
  cases need a temporary "worse" intermediate state before the fix lands.
  The working code does a small (depth ≤4) BFS over
  `{algorithm A, algorithm B} × {0-3 U pre-rotations}` instead.
- **A bare U pre-rotation before a PLL-edges algorithm will permanently
  misplace already-solved PLL corners** if it isn't undone afterward (a
  plain "rotate U, apply algorithm" leaves corners rotated out of place).
  The fix conjugates it: `U^k, algorithm, U'^k`, since the edge algorithm
  provably never moves corner *positions* (verified with unique per-sticker
  tags), so the trailing `U'^k` cleanly restores them.

## Current state / possible follow-ups

The solver produces a correct but non-optimal solution (beginner's method,
typically ~100-150 moves) — this was a correctness fix, not a move-count
optimization. If shorter solutions are ever wanted, that's a from-scratch
different project (e.g. Kociemba's two-phase algorithm), not a tweak to
this code.
