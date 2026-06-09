# ArrostiTetris Animation Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create a single `index.html` that animates the SVG poster pieces falling one-by-one like Tetris, curving toward a mouse-following skewer.

**Architecture:** Single HTML file with embedded SVG (pieces moved to `<defs>` as centered templates), JS vanilla with `requestAnimationFrame`. Pieces spawn at random X above the viewport, fall at constant speed while curving toward the skewer X, accumulate on the skewer, then loop.

**Tech Stack:** HTML5, SVG, vanilla JS (no libraries)

---

## File Structure

- **Create:** `index.html` — The entire application (SVG + styles + JS)
- **Modify (reference):** `asset/poster.svg` — Source SVG read for coordinates

`index.html` responsibilities:
- SVG with the game background (sfondo), stecchino, hearts, UI text
- Each pezzo in `<defs>` wrapped to be centered at (0,0) for uniform `translate()` usage
- `<script>` with all animation logic

---

### Task 1: Create `index.html` with modified SVG (pieces in `<defs>`)

- [ ] **Step 1: Copy `asset/poster.svg` content into new `index.html`, wrap with HTML boilerplate**

```bash
cp asset/poster.svg index.html
```

- [ ] **Step 2: Add HTML document wrapper around the SVG**

```html
<!DOCTYPE html>
<html lang="it">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>ArrostiTetris</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body {
    background: #1a1a1a;
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 100vh;
  }
  svg {
    width: 595.28px;
    height: 841.89px;
    max-height: 98vh;
    cursor: none;
  }
</style>
</head>
<body>
  <!-- SVG content here -->
<script>
// JS animation code
</script>
</body>
</html>
```

- [ ] **Step 3: Move each `pezzo_N` group into `<defs>`, wrap with centering transform**

For each pezzo, create a template `<g id="t_pezzo_N">` inside `<defs>` with a nested `<g>` that translates the piece so its center is at (0,0). Center X for all pieces = 197.5. Center Y per piece:

| Piece  | Original Y range | Center Y | Translate offset |
|--------|-----------------|----------|------------------|
| pezzo_7 | 267–317        | 292      | `translate(-197.5, -292)` |
| pezzo_6 | 319–341        | 330      | `translate(-197.5, -330)` |
| pezzo_5 | 343–393        | 368      | `translate(-197.5, -368)` |
| pezzo_4 | 395–417        | 406      | `translate(-197.5, -406)` |
| pezzo_3 | 419–469        | 444      | `translate(-197.5, -444)` |
| pezzo_2 | 470–520        | 495      | `translate(-197.5, -495)` |
| pezzo_1 | 522–544        | 533      | `translate(-197.5, -533)` |

Result in `<defs>`:

```svg
<defs>
  <style>...</style>
  <g id="t_pezzo_7">
    <g transform="translate(-197.5, -292)">
      <path class="cls-9" d="M222.18,269.06..."/>
      <polygon class="cls-15" points="222.18 272.75..."/>
    </g>
  </g>
  <!-- repeat for pezzo_6 through pezzo_1 -->
</defs>
```

- [ ] **Step 4: Remove the original pezzo_N groups from the SVG body** (lines 176–203 in original SVG). The `pezzo_1` through `pezzo_7` `<g>` elements should be deleted from the main SVG body since they'll be rendered dynamically.

- [ ] **Step 5: Wrap stecchino in an identifiable group**

```svg
<g id="moving-group">
  <g id="stecchino">
    <path class="cls-6" d="..."/>
    <polygon class="cls-6" points="..."/>
  </g>
  <g id="landed-pieces"></g>
</g>
```

- [ ] **Step 6: Open `index.html` in browser to verify the SVG renders correctly** (static state, no pieces yet)

---

### Task 2: Add animation JS — state, mouse tracking, and game loop

- [ ] **Step 1: Add initialization code and state variables**

```js
const PIECE_TYPES = ['t_pezzo_1', 't_pezzo_2', 't_pezzo_3', 't_pezzo_4', 't_pezzo_5', 't_pezzo_6', 't_pezzo_7'];
const PIECE_HEIGHT = 38; // vertical spacing between pieces on skewer
const BASE_LAND_Y = 530; // first piece landing Y
const FALL_SPEED = 2; // pixels per frame
const ATTRACTION_FORCE = 0.05; // how fast pieces curve toward skewer
const GAME_AREA_LEFT = 51;
const GAME_AREA_RIGHT = 344;
const STECCHINO_CENTER_X = 197.5;
const SPAWN_Y = -100;

let stecchinoX = STECCHINO_CENTER_X;
let pieces = []; // { type: string, x: number, y: number, landed: boolean }
let currentPiece = null; // { type, x, y, targetSlot }
let pieceQueue = [];
let landedCount = 0;
let isRunning = true;

const svg = document.querySelector('#Poster');
const movingGroup = document.getElementById('moving-group');
const landedContainer = document.getElementById('landed-pieces');
```

- [ ] **Step 2: Mouse tracking on the SVG element**

```js
svg.addEventListener('mousemove', (e) => {
  const rect = svg.getBoundingClientRect();
  const scaleX = 595.28 / rect.width;
  const svgX = (e.clientX - rect.left) * scaleX;
  stecchinoX = Math.max(GAME_AREA_LEFT + 25, Math.min(GAME_AREA_RIGHT - 25, svgX));
});

svg.addEventListener('mouseleave', () => {
  // keep last position
});
```

- [ ] **Step 3: Piece spawning logic**

```js
function shuffle(arr) {
  for (let i = arr.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [arr[i], arr[j]] = [arr[j], arr[i]];
  }
  return arr;
}

function fillQueue() {
  pieceQueue = shuffle([...PIECE_TYPES]);
}

function spawnNextPiece() {
  if (pieceQueue.length === 0) {
    fillQueue();
    landedCount = 0;
  }
  const type = pieceQueue.pop();
  const spawnX = GAME_AREA_LEFT + 25 + Math.random() * (GAME_AREA_RIGHT - GAME_AREA_LEFT - 50);
  currentPiece = {
    type: type,
    x: spawnX,
    y: SPAWN_Y,
    targetSlot: landedCount
  };
}

fillQueue();
spawnNextPiece();
```

- [ ] **Step 4: Animation loop with `requestAnimationFrame`**

```js
function update() {
  if (!currentPiece) return;

  // Move piece down
  currentPiece.y += FALL_SPEED;

  // Curve toward stecchino
  const dx = stecchinoX - currentPiece.x;
  currentPiece.x += dx * ATTRACTION_FORCE;

  // Check if reached landing Y
  const targetY = BASE_LAND_Y - currentPiece.targetSlot * PIECE_HEIGHT;
  if (currentPiece.y >= targetY) {
    // Land the piece
    currentPiece.y = targetY;
    currentPiece.x = stecchinoX;
    pieces.push({ ...currentPiece, landed: true });
    landedCount++;

    // Render landed piece
    const use = document.createElementNS('http://www.w3.org/2000/svg', 'use');
    use.setAttributeNS('http://www.w3.org/1999/xlink', 'href', `#${currentPiece.type}`);
    use.setAttribute('transform', `translate(${stecchinoX}, ${targetY})`);
    landedContainer.appendChild(use);

    currentPiece = null;

    // Spawn next after delay
    setTimeout(spawnNextPiece, 500);
  }

  render();
  requestAnimationFrame(update);
}
```

- [ ] **Step 5: Render the falling piece**

```js
let fallingUse = null;

function render() {
  // Update moving group (stecchino + landed pieces)
  const currentOffset = stecchinoX - STECCHINO_CENTER_X;
  movingGroup.setAttribute('transform', `translate(${currentOffset}, 0)`);

  // Update landed pieces positions (they move with stecchino)
  const uses = landedContainer.querySelectorAll('use');
  for (let i = 0; i < uses.length; i++) {
    const piece = pieces[i];
    if (piece) {
      uses[i].setAttribute('transform', `translate(${stecchinoX}, ${piece.y})`);
    }
  }

  // Render falling piece
  if (currentPiece) {
    if (!fallingUse) {
      fallingUse = document.createElementNS('http://www.w3.org/2000/svg', 'use');
      fallingUse.setAttributeNS('http://www.w3.org/1999/xlink', 'href', `#${currentPiece.type}`);
      svg.appendChild(fallingUse);
    } else {
      fallingUse.setAttributeNS('http://www.w3.org/1999/xlink', 'href', `#${currentPiece.type}`);
    }
    fallingUse.setAttribute('transform', `translate(${currentPiece.x}, ${currentPiece.y})`);
  } else if (fallingUse) {
    fallingUse.remove();
    fallingUse = null;
  }
}
```

- [ ] **Step 6: Start the loop and handle page visibility**

```js
function start() {
  update();
}

document.addEventListener('visibilitychange', () => {
  isRunning = !document.hidden;
  if (isRunning) update();
});

start();
```

- [ ] **Step 7: Open in browser and verify** — pieces should spawn, fall, curve toward stecchino, land, and loop

---

### Task 3: Tuning and polish

- [ ] **Step 1: Verify piece stacking looks correct** — adjust `PIECE_HEIGHT`, `BASE_LAND_Y`, `FALL_SPEED`, `ATTRACTION_FORCE` constants as needed

- [ ] **Step 2: Verify stecchino follows mouse smoothly** — test at various speeds, verify pieces follow

- [ ] **Step 3: Test the loop** — wait for all 7 pieces to land, verify new cycle starts with shuffled order

- [ ] **Step 4: Open `index.html` in browser, confirm everything works**

---

### Self-Review

- [ ] All spec requirements covered: pieces fall from random top positions, one at a time, curve toward stecchino, accumulate, stecchino follows mouse, infinite loop
- [ ] No placeholders, TODOs, or ambiguous steps
- [ ] All file paths absolute
- [ ] All code blocks contain actual working code
