# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JS Tetris. No dependencies, no build step, no package.json, no test suite. Three files: `index.html`, `style.css`, `game.js`.

## Running

Open `index.html` directly in a browser, or serve statically:

```bash
npx serve .
# or
python3 -m http.server 8000
```

There is no build/lint/test command — this repo has none configured.

## Architecture

All game logic lives in `game.js` (~300 lines, single file, no modules). It renders to two `<canvas>` elements defined in `index.html`: `#board` (300×600, the playfield) and `#next-canvas` (120×120, next-piece preview).

Key structures:
- `board`: a `ROWS × COLS` matrix; each cell is `0` (empty) or a color index `1-7` identifying a locked piece.
- `PIECES` / `COLORS`: parallel arrays indexed by piece type (1-7) defining shapes and colors. Piece shapes are square matrices; `rotateCW` rotates via transpose + row-reverse.
- `current` / `next`: the falling piece and the queued next piece, each `{ type, shape, x, y }`.

Core flow: `init()` sets up state and starts `requestAnimationFrame(loop)`. `loop(ts)` accumulates elapsed time (`dropAccum`) and advances the piece down one row once `dropInterval` is exceeded, otherwise calls `lockPiece()` (which merges into `board`, clears lines, and spawns the next piece). `draw()` redraws the grid, locked board, ghost piece, and current piece every frame.

Collision (`collide`), rotation with wall kicks (`tryRotate`, kick offsets `[0,-1,1,-2,2]`), line clearing (`clearLines`, bottom-to-top scan with splice/unshift), and scoring (`LINE_SCORES` table × level, plus soft/hard drop bonuses) are all self-contained functions near the top of `game.js` — read them together to understand a change to game rules.

Level/speed: level increases every 10 lines; `dropInterval = max(100, 1000 - (level-1)*90)`.

Input is a single `keydown` listener switching on `e.code` (arrows, `KeyX` to rotate, `Space` for hard drop, `KeyP` to pause).

## Tunable constants (top of game.js)

`COLS`, `ROWS`, `BLOCK` (cell px size), `COLORS`, `LINE_SCORES`, initial `dropInterval`. If `COLS`/`ROWS`/`BLOCK` change, update the `#board` canvas `width`/`height` in `index.html` to match (`COLS×BLOCK` by `ROWS×BLOCK`).
