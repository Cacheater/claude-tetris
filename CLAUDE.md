# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JavaScript Tetris. No dependencies, no build step, no package manager — just `index.html`, `style.css`, and `game.js`.

## Running the game

Open `index.html` directly in a browser, or serve it locally (needed if the browser blocks local file access):

```bash
python3 -m http.server 8000
# or
npx serve .
```

There is no test suite, linter, or build/bundle process in this repo.

## Architecture

Everything lives in three files with no module system — `game.js` is a single script loaded directly by `index.html`, relying on global `const`/`let` bindings and top-level function declarations.

- **Board model**: `board` is a `ROWS × COLS` (20×10) matrix of numbers. `0` = empty, `1`–`7` = a color index identifying which piece type occupies that cell (see `COLORS`/`PIECES`).
- **Pieces**: the 7 standard tetrominoes are hardcoded as square matrices in `PIECES`. `current` and `next` are piece objects `{ type, shape, x, y }`. Rotation is `rotateCW` (transpose + reverse), not a lookup table of rotation states.
- **Collision & wall kicks**: `collide(shape, ox, oy)` checks board bounds and existing fixed blocks. `tryRotate()` rotates and, on collision, retries with x-offsets `[0, -1, 1, -2, 2]` before giving up (simplified wall-kick, not the full SRS kick table).
- **Game loop**: `loop(ts)` runs via `requestAnimationFrame`, accumulating elapsed time in `dropAccum` and dropping the piece one row once `dropAccum >= dropInterval`. Pausing/resuming cancels/restarts the `requestAnimationFrame` chain (`animId`) rather than gating logic inside the loop.
- **Locking & line clears**: `lockPiece()` → `merge()` writes the piece into `board`, then `clearLines()` scans bottom-up, splicing out full rows and unshifting empty ones at the top.
- **Scoring/leveling**: `LINE_SCORES = [0,100,300,500,800]` multiplied by `level`; hard drop adds 2 pts/row dropped, soft drop 1 pt/row. `level = floor(lines / 10) + 1`; `dropInterval = max(100, 1000 - (level-1)*90)`.
- **Rendering**: `draw()` clears and redraws the whole board canvas every frame (grid, locked blocks, ghost piece via `ghostY()` at `globalAlpha = 0.2`, then the current piece). `drawNext()` renders the next-piece preview on a separate canvas/context.
- **State**: nearly all mutable game state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, timing vars) is declared as bare module-level `let`s and mutated directly by functions — there's no state container or event system. `init()` resets all of it and is also wired to the restart button.

If you change `COLS`, `ROWS`, or `BLOCK` in `game.js`, also update the `<canvas id="board">` `width`/`height` in `index.html` to match (`COLS × BLOCK`, `ROWS × BLOCK`).
