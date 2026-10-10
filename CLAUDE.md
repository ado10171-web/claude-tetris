# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JS Tetris (HTML5 Canvas). No `package.json`, bundler, linter, or tests — there is no build/lint/test step. The README and UI text are in Spanish; keep new user-facing strings consistent with that.

## Running

Open `index.html` directly, or serve statically (e.g. `python -m http.server 8000`) and visit `http://localhost:8000`.

## Architecture

All logic lives in `game.js` (a single classic script loaded by `index.html`, using global state, `'use strict'`). It binds to DOM ids defined in `index.html` (`board`, `next-canvas`, `score`, `lines`, `level`, `overlay`, `overlay-title`, `overlay-score`, `restart-btn`) — renaming an id requires updating both files.

Key points that span the code:

- **State** is module-level `let` variables (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, `animId`, …), all reset in `init()`, which is also the restart handler.
- **Piece encoding**: the board stores `0` or a color/type index 1–7; `PIECES` and `COLORS` are indexed by that same number, and the piece matrices contain their own index as the filled value. Adding a piece means updating both arrays and the `* 7` in `randomPiece()`.
- **Piece lifecycle**: `lockPiece()` → `merge()` → `clearLines()` → `spawn()`. `spawn()` calls `endGame()` if the new piece collides immediately. Gravity (in `loop`), `softDrop`, and `hardDrop` all funnel into `lockPiece()`.
- **Loop**: `requestAnimationFrame` `loop` accumulates `dropAccum` against `dropInterval`; `togglePause`/`endGame`/`init` manage it via `cancelAnimationFrame(animId)`. Level/speed are recomputed inside `clearLines()`.
- **Rotation** uses `rotateCW` plus horizontal-only kicks `[0, -1, 1, -2, 2]` in `tryRotate()` (no floor kicks, no SRS tables).
- **Rendering** is split: `draw()` (main board, grid, ghost via `ghostY()`, current piece) and `drawNext()` (preview canvas); both use `drawBlock`. The HUD is updated by `updateHUD()`.
- **Canvas size is hard-coded in HTML**: `#board` is 300×600 and must equal `COLS*BLOCK` × `ROWS*BLOCK`; `#next-canvas` is 120×120 (4×4 cells at 30px). Changing `COLS`/`ROWS`/`BLOCK` requires editing `index.html` too (and CSS if layout depends on it).
