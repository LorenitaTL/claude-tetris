# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the game

No build step or dependencies. Open directly:

```bash
open index.html          # macOS
python3 -m http.server 8000  # then visit http://localhost:8000
```

## Architecture

Three files, zero dependencies:

- **`index.html`** — DOM structure: a `<canvas id="board">` (300×600 px) for the board and a `<canvas id="next-canvas">` (120×120 px) for the preview. The `#overlay` div handles both PAUSE and GAME OVER states via a `hidden` CSS class toggle.
- **`style.css`** — Dark/retro arcade theme. Uses `backdrop-filter` on the overlay and monospaced fonts for the HUD.
- **`game.js`** — All game logic (~305 lines, `'use strict'`).

### game.js internals

The board is a `ROWS × COLS` matrix where `0` = empty and `1–7` = piece color index (maps into `COLORS[]`).

Key functions and how they connect:
- `init()` → resets all state, calls `spawn()`, starts `requestAnimationFrame(loop)`
- `loop(ts)` → accumulates delta time; when `dropAccum ≥ dropInterval`, moves piece down or calls `lockPiece()`
- `lockPiece()` → `merge()` (writes piece to board) → `clearLines()` → `spawn()`
- `spawn()` → promotes `next` to `current`, generates new `next`; collision at spawn triggers `endGame()`
- `tryRotate()` → `rotateCW()` + wall-kick offsets `[0, -1, 1, -2, 2]`
- `ghostY()` → projects current piece straight down to find landing row; rendered at `globalAlpha = 0.2`

Speed formula: `dropInterval = Math.max(100, 1000 − (level − 1) × 90)` ms. Level increases every 10 lines.

### Tunable constants (top of game.js)

| Constant | Default | Note |
|---|---|---|
| `COLS` / `ROWS` | 10 / 20 | If changed, update canvas `width`/`height` in `index.html` to `COLS×BLOCK` / `ROWS×BLOCK` |
| `BLOCK` | 30 | Pixel size per cell |
| `COLORS` | 7 colors | Index 0 is `null` (empty); indices 1–7 map to piece types |
| `LINE_SCORES` | `[0,100,300,500,800]` | Multiplied by current level |
