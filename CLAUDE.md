# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the game

No build step — open directly in a browser:

```bash
xdg-open index.html          # Linux
# or serve with any static server:
python3 -m http.server 8000  # then open http://localhost:8000
```

## Architecture

Three files, no dependencies, no bundler:

- **`index.html`** — DOM structure: `<canvas id="board">` (300×600 px) for the playfield, `<canvas id="next-canvas">` (120×120 px) for the next-piece preview, a sidebar panel with score/lines/level displays, and a hidden overlay div reused for both PAUSE and GAME OVER states.
- **`style.css`** — Dark/retro arcade theme using flexbox and CSS variables.
- **`game.js`** — All game logic (~300 lines, `'use strict'`).

### game.js internals

**State** (module-level `let` vars): `board` (10×20 matrix of color indices 0–7), `current`/`next` (piece objects with `{type, shape, x, y}`), `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, `dropAccum`, `lastTime`, `animId`.

**Key functions and their roles:**

| Function | Role |
|---|---|
| `init()` | Resets all state, starts the `requestAnimationFrame` loop |
| `loop(ts)` | Game loop: accumulates `dt`, drops piece when `dropAccum >= dropInterval`, calls `draw()` |
| `collide(shape, ox, oy)` | Bounds + overlap check against `board` |
| `tryRotate()` | Rotates `current` CW with wall kicks `[0, -1, 1, -2, 2]` |
| `lockPiece()` | Calls `merge()` → `clearLines()` → `spawn()` |
| `spawn()` | Promotes `next` to `current`; triggers `endGame()` if it immediately collides |
| `clearLines()` | Scans bottom-up; splices complete rows and unshifts empty ones; updates score/level/speed |
| `ghostY()` | Projects `current` downward until collision to find landing row |
| `draw()` | Clears canvas, draws grid + locked board cells + ghost (α=0.2) + current piece |

**Speed formula**: `dropInterval = Math.max(100, 1000 − (level − 1) × 90)` ms. Level increases every 10 lines.

**Scoring**: `LINE_SCORES = [0, 100, 300, 500, 800]` × level. Hard drop adds 2 pts/row; soft drop adds 1 pt/row.

### Tunable constants (top of game.js)

`COLS`, `ROWS`, `BLOCK` (cell size in px), `COLORS` (array indexed 1–7), `LINE_SCORES`. If you change `COLS`, `ROWS`, or `BLOCK`, update the `width`/`height` attributes on `<canvas id="board">` in `index.html` to match (`COLS × BLOCK` and `ROWS × BLOCK`).
