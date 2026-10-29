# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A classic Tetris implementation in vanilla JavaScript, HTML5 Canvas, and CSS. No dependencies, no build step, no package.json — just three files that cooperate: `index.html`, `style.css`, `game.js`.

## Running the game

There is no build/test/lint tooling. To run it, open `index.html` directly or serve the directory statically:

```bash
xdg-open index.html          # open directly
python3 -m http.server 8000  # or serve locally, then visit http://localhost:8000
```

Verify changes by actually playing in a browser — there are no automated tests.

## Architecture

All game logic lives in `game.js` (single file, no modules). The three files divide responsibility as:

- `index.html` — DOM structure: a `300×600` `<canvas id="board">` for the playfield, a `120×120` `<canvas id="next-canvas">` for the next-piece preview, HUD spans (`#score`, `#lines`, `#level`), and an `#overlay` used for both PAUSE and GAME OVER.
- `style.css` — dark/retro arcade visual theme.
- `game.js` — everything else, built around a small set of global `let` state variables (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, etc.) rather than a class or module pattern.

Key mechanics in `game.js`:

- **Board model**: `ROWS × COLS` matrix where each cell is `0` (empty) or a color index `1–8` identifying which piece locked there.
- **Pieces**: defined as square matrices in `PIECES`. Index 8 is the "tuerca" (nut): a 3×3 ring with an empty center that leaves an enclosed hole when locked, as an extra challenge. Rotation is done via `rotateCW` (transpose + reverse), not by storing pre-rotated states.
- **Collision** (`collide`): checks board bounds and overlap with locked cells.
- **Wall kicks** (`tryRotate`): after rotating, tries offsets `[0, -1, 1, -2, 2]` until a non-colliding position is found.
- **Game loop** (`loop`): driven by `requestAnimationFrame`; accumulates elapsed time in `dropAccum` and advances the piece one row once `dropInterval` is exceeded.
- **Line clearing** (`clearLines`): scans bottom-up, splices full rows out and unshifts empty rows at the top.
- **Scoring**: `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level`; hard drop adds 2 points/cell dropped, soft drop adds 1 point/row.
- **Leveling/speed**: level increases every 10 lines; `dropInterval = max(100, 1000 - (level - 1) * 90)` ms.
- **Ghost piece** (`ghostY`): projects the current piece straight down and renders it at `globalAlpha = 0.2`.

Flow: `init()` builds the board, seeds `next`, calls `spawn()` (which promotes `next` to `current` and generates a new `next`), then starts the `requestAnimationFrame` loop. `spawn()` triggers `endGame()` if the new piece immediately collides. Keyboard input (`keydown`) drives movement/rotation/drop/pause directly against the global state.

## Tunable constants (top of `game.js`)

`COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, `dropInterval` (initial). If `COLS`, `ROWS`, or `BLOCK` change, update the `width`/`height` of `<canvas id="board">` in `index.html` to match (`COLS × BLOCK` and `ROWS × BLOCK`).
