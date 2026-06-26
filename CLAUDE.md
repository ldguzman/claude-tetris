# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the game

No build step required. Open `index.html` directly or serve it locally:

```bash
python3 -m http.server 8000   # then open http://localhost:8000
open index.html               # macOS — open directly
```

## Architecture

Three files, no dependencies, no bundler:

- **`index.html`** — DOM skeleton: `<canvas id="board">` (300×600 px) for the playfield, `<canvas id="next-canvas">` (120×120 px) for the next-piece preview, HUD elements (`#score`, `#lines`, `#level`), and a single `#overlay` div reused for both PAUSE and GAME OVER states.
- **`style.css`** — dark/retro theme; no classes are added/removed by JS except `overlay.classList.toggle('hidden')`.
- **`game.js`** — all game logic (~305 lines, `'use strict'`, no modules).

### Key data model (`game.js`)

| Variable | Description |
|---|---|
| `board` | `ROWS×COLS` number array; `0` = empty, `1–7` = piece color index |
| `current` | `{ type, shape, x, y }` — the active falling piece |
| `next` | Same shape as `current`; becomes `current` on `spawn()` |

### Core function flow

```
init() → spawn() → requestAnimationFrame(loop)
loop(ts): accumulates dt, drops piece or calls lockPiece() on collision, then draw()
lockPiece(): merge() → clearLines() → spawn()
spawn(): if new piece immediately collides → endGame()
```

### Canvas sizing constraint

The board canvas is sized as `COLS × BLOCK` × `ROWS × BLOCK`. If you change `COLS`, `ROWS`, or `BLOCK` in `game.js`, update `width`/`height` on `<canvas id="board">` in `index.html` to match.

### Rotation & wall kicks

`rotateCW(shape)` computes clockwise rotation via transpose+reverse. `tryRotate()` attempts the rotation with horizontal offsets `[0, -1, 1, -2, 2]` and applies the first non-colliding kick.

### Speed formula

`dropInterval = Math.max(100, 1000 - (level - 1) * 90)` ms. Level increases every 10 lines.
