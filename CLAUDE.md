# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla Tetris implementation. No dependencies, no build step, no package.json — three files (`index.html`, `style.css`, `game.js`) cooperate directly via a `<script>` tag.

## Running the game

No install/build required.

```bash
open index.html                # macOS, direct file open
python3 -m http.server 8000    # or: npx serve .   /   php -S localhost:8000
```

There are no tests, linter, or bundler configured in this repo.

## Architecture

All game logic lives in `game.js` (~300 lines), organized around a small set of cooperating pieces:

- **Board model**: `board` is a `ROWS × COLS` matrix; each cell is `0` (empty) or a color index `1–7` identifying which piece locked there.
- **Pieces**: `PIECES` defines the 7 tetrominoes as square matrices. Rotation (`rotateCW`) is done via transpose + row-reverse, not by storing rotation states.
- **Collision** (`collide`): checks board bounds and overlap with locked cells; used for movement, rotation, and computing the ghost piece.
- **Wall kicks** (`tryRotate`): after rotating, tries offsets `[0, -1, 1, -2, 2]` columns until a non-colliding position is found, else the rotation is discarded.
- **Game loop** (`loop`): driven by `requestAnimationFrame`; accumulates elapsed time in `dropAccum` and advances the piece one row once `dropInterval` is exceeded.
- **Line clearing** (`clearLines`): scans bottom-to-top, splices full rows out and unshifts empty rows at the top; scoring uses `LINE_SCORES` (`[0,100,300,500,800]`) multiplied by `level`.
- **Level/speed**: level increases every 10 lines; `dropInterval = max(100, 1000 - (level-1)*90)` ms.
- **Ghost piece** (`ghostY`): projects the current piece straight down to its landing row, drawn at `globalAlpha = 0.2`.

Control flow: `init()` builds the board and starts the loop → `loop()` ticks gravity and calls `draw()` each frame → `lockPiece()` (called from gravity, soft drop, or hard drop) merges the piece into the board, clears lines, and spawns the next one → `spawn()` promotes `next` to `current` and generates a new `next`; if the new piece immediately collides, `endGame()` fires.

Tunable constants are all at the top of `game.js` (`COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, initial `dropInterval`). If `COLS`, `ROWS`, or `BLOCK` change, the `<canvas id="board">` `width`/`height` in `index.html` must be updated to match (`COLS × BLOCK`, `ROWS × BLOCK`).

README.md is in Spanish and contains the same architectural walkthrough in more detail.
