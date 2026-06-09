# ArrostiTetris — Project Summary

## Game
Tetris-like pieces (carne/verdure skewers) fall into a moving skewer. Pieces impale on contact with the skewer and slide to their target position. 7 pieces = win.

## Mechanics
- **Skewer**: Moves with mouse X, horizontal SVG game area
- **Pieces**: 7 normal pieces (t_pezzo_1 through t_pezzo_7) in a fixed queue, each with a target Y position on the skewer
- **Red pieces** (25% chance): Same size as the next normal piece in queue. If impaled → lose a heart + red piece replaces the normal in queue (no overlap). If missed → disappears, normal arrives later. Max 3 reds.
- **Miss**: Piece falls past game area bottom → respawns at top
- **FALL_SPEED**: 5, **SLIDE_SPEED**: 4
- **Win**: All 7 skewer slots filled (normals + reds)
- **Lose**: 3 hearts lost (3 reds caught)
- **Hearts**: cuore_1/2/3 displayed, hidden when lost
- **Pause**: Click start button during play
- **Confetti**: 8 falling rectangles on win

## Queue System
- Initial: [`t_pezzo_7`, `t_pezzo_6`, ..., `t_pezzo_1`] (pop order: 1→7)
- Reds peek at the last item for size/targetY, don't pop
- Reds impaled → replacesType removed from queue via splice
- Invariant: pieces.length + queue.length = 7
- No queue recycling (single play-through)

## SVG Templates
- Normal pieces: `t_pezzo_1..7` (class `cls-9` outline, `cls-4` fill)
- Red pieces: `t_pezzo_rosso_1..7` (class `cls-9` outline, `cls-1` red fill)
- Skewer: `#stecchino` in `#moving-group` with `#landed-pieces`
- Template transform: `translate(-197.5, -Y)` for each piece type
- Pieces rendered at `translate(X, Y)` on the skewer with offset from center

## Key Constants
- `GAME_AREA_BOTTOM`: 573.66
- `RED_CHANCE`: 0.25
- Piece target Ys: pezzo_1→533 down to pezzo_7→292
- Collision: x diff < 28 && y is near 210

## Architecture
- Single SVG (index.html), SVG `<use>` elements for piece rendering
- Falling piece: detached `<use>` in `#game-area` (no `#moving-group` transform)
- Landed pieces: `#landed-pieces` inside `#moving-group` (skewer transform applies)

## Template Colors (SVG classes)
- cls-1: #4c442b (brown/sfondo)
- cls-4: #ffffff (white fill - piece interior)
- cls-9: #29170f (dark brown - piece outline)
- Pezzi rossi: same outline, cls-1 fill (brown = red)
