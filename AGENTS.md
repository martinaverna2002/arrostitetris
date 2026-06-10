# ArrostiTetris — Agent Guide

## Project Overview
Single-file SVG game (`index.html`) — pieces fall from top and must be impaled on a horizontally moving skewer. 7 pieces total (pezzo_1 → pezzo_7). Pieces slide to fixed Y positions on the skewer. Win when all 7 are landed.

## File Structure
```
arrostitetris/
├── index.html          # Everything: SVG + CSS + JS
├── GEMINI.md           # Project summary (auto-generated)
└── AGENTS.md           # This file
```

## Game Mechanics

### Pieces & Queue
- **7 piece types**: `t_pezzo_1` through `t_pezzo_7` (fixed-size templates in SVG `<defs>`)
- **Queue**: filled once at game start (`fillQueue()`), reversed order: pops pezzo_1 first, then 2…7
- **Target Y positions** (`PIECE_LAND_Y`):
  - pezzo_1 → 533 (bottom), pezzo_2 → 495, pezzo_3 → 444, pezzo_4 → 406,
  - pezzo_5 → 368, pezzo_6 → 330, pezzo_7 → 292 (top)
- **Missed piece**: if it falls past `GAME_AREA_BOTTOM` (573.66), it **respawns at top** with new random X — same piece retries until impaled
- **No overlap**: each piece has a unique fixed targetY, so they never overlap on the skewer

### Impalement Flow
1. Piece falls at `FALL_SPEED` (3)
2. Collision detected when near skewer tip (X diff < 28, Y near 210)
3. Piece is added to `pieces[]` array, marked as `.sliding = true`
4. Piece slides down at `SLIDE_SPEED` (4) toward its `targetY`
5. When sliding completes, `spawnNextPiece()` is called to spawn the next piece from queue
6. When queue is empty → `winGame()`

### Win/Lose
- **Win**: all 7 pieces impaled and slid to position (`pieces.length === 7` triggers win on next `spawnNextPiece()` call)
- **Lose**: no lose condition currently (hearts are decorative only, kept at user request)
- **Win text**: "YOU WIN" appears + confetti (8 colored rectangles falling with CSS animation)
- **Auto-reset**: after win, game resets to idle after 5 seconds

### Controls
- **Mouse**: moves skewer horizontally (constrained to game area bounds)
- **Start button**: click to start countdown (3→2→1→GO), or pause/resume during play
- **Pause**: click start button during play → shows pause icon (two vertical bars)
- **Visibility change**: auto-pauses when tab hidden

## SVG Architecture

### Layer Structure
```
#Poster (root SVG, 595.28×841.89)
├── <defs>
│   ├── t_pezzo_1..7          # Normal piece templates (cls-9 outline, cls-4 fill)
│   └── clipPath #white-rect-clip
├── #sfondo                   # Static background (score text, buttons, hearts)
├── #game-area                # Clipped to white rectangle
│   ├── #moving-group         # Translates with skewer X
│   │   ├── #stecchino        # Skewer path + tip polygon
│   │   └── #landed-pieces    # Container for impaled pieces (<use> elements)
│   ├── #countdown            # "3", "2", "1", "GO"
│   ├── #pause-icon           # Two vertical bars
│   ├── #win-text             # "YOU WIN"
│   └── #confetti-container   # Falling rectangles
└── #cuore_1, #cuore_2, #cuore_3  # Decorative hearts
```

### Piece Rendering
- **Template**: each piece is a `<g>` with `transform="translate(-197.5, -Y)"` where Y is the piece's natural center
- **Falling piece**: `<use href="#t_pezzo_N">` in `#game-area` at `translate(X, Y)`
- **Landed piece**: `<use href="#t_pezzo_N">` in `#landed-pieces` at `translate(STECCHINO_CENTER_X + offset, Y)`
- The landed piece stays inside `#moving-group`, so it moves with the skewer

## Key Constants
| Constant | Value | Purpose |
|----------|-------|---------|
| `FALL_SPEED` | 3 | Falling speed |
| `SLIDE_SPEED` | 4 | Post-impalement slide speed |
| `GAME_AREA_LEFT` | 51.05 | Game area left bound |
| `GAME_AREA_RIGHT` | 344.18 | Game area right bound |
| `GAME_AREA_BOTTOM` | 573.66 | Bottom bound (piece respawns here) |
| `STECCHINO_CENTER_X` | 197.5 | Skewer tip X position |
| `SPAWN_Y` | 49.95 | Piece spawn Y (top of game area) |

## State Machine
```javascript
STATE = { IDLE: 0, COUNTDOWN: 1, PLAYING: 2, PAUSED: 3, WIN: 4 }
```
- `IDLE` → click start → `COUNTDOWN`
- `COUNTDOWN` → 3-2-1 → `PLAYING`
- `PLAYING` → click start → `PAUSED`
- `PAUSED` → click start → `PLAYING`
- `PLAYING` → 7 pieces landed → `WIN`
- `WIN` → click start or 5s timeout → `IDLE`

## Data Structures
- `pieces[]`: array of landed piece objects `{ type, x, y, targetY, sliding, use, offset }`
- `currentPiece`: the falling piece object (or null)
- `pieceQueue[]`: remaining pieces to spawn, filled once at start
- `stecchinoX`: current skewer X position (mouse-controlled)

## Important Notes for Agents
1. **Single file**: all code is in `index.html` — SVG, CSS, and JS together
2. **No build step**: just open in browser
3. **Queue invariant**: `pieces.length + queue.length === 7` during play (each pop → one piece on skewer)
4. **No recycling**: queue is filled once; if empty and pieces < 7, it's an edge case (shouldn't happen in normal play)
5. **Hearts are decorative**: `cuore_1/2/3` exist in SVG but no JS logic removes them currently
6. **Do NOT add**: red/rotten pieces, lives system, queue recycling, piece randomization — user wants simple 1-7 sequential gameplay
7. **Template transforms**: pieces use `translate(-197.5, -targetY)` in `<defs>`, then `translate(X, Y)` on `<use>` — this is critical for correct positioning
