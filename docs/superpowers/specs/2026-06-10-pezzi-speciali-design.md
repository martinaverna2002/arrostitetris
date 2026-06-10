# Design — Pezzi Speciali "Muffa" e Sistema Vite

**Data:** 2026-06-10  
**Progetto:** ArrostiTetris  
**Stato:** Approvato

---

## 1. Obiettivo

Aggiungere al gioco **pezzi speciali "a muffa"** che il giocatore deve schivare, e un **sistema di vite** (3 cuori) che si perde se un pezzo speciale colpisce lo stecchino. La condizione di vittoria resta infilzare tutti e 7 i pezzi normali.

---

## 2. Macchina a Stati

| Stato | Descrizione |
|---|---|
| `IDLE` | Gioco in pausa, attende click su Start |
| `COUNTDOWN` | Conto alla rovescia 3→2→1→GO |
| `PLAYING` | Gioco attivo: pezzi cadono, vite attive |
| `PAUSED` | Pausa manuale (click Start) |
| `WIN` | 7 pezzi normali infilzati → "YOU WIN", confetti, 5s poi `IDLE` |
| `GAME_OVER` | **NUOVO** — 3 vite perse → "GAME OVER", flash rosso, shake, 3s poi `IDLE` |

Transizioni:
- `PLAYING → GAME_OVER`: quando `lives === 0`
- `GAME_OVER → IDLE`: dopo 3s automaticamente
- `WIN → IDLE`: dopo 5s automaticamente

---

## 3. Sistema di Vite (Cuori)

- **Vite iniziali**: 3
- **Visuali**: i 3 cuori SVG esistenti (`#cuore_1`, `#cuore_2`, `#cuore_3`) in alto a destra del poster
- **Perdita vita**: al contatto stecchino–pezzo-speciale → il cuore corrispondente diventa `visibility: hidden`
- **Reset**: alla nuova partita, tutti e 3 i cuori tornano visibili

---

## 4. Pezzi Speciali "a Muffa"

### 4.1 Spawn
- All’avvio della partita viene estratta una **probabilità base** compresa tra **30% e 60%** (decide quanto la partita sarà "infestata").
- Ogni volta che deve apparire un nuovo pezzo in caduta, si tira un dado: se esce sotto la soglia → pezzo **speciale**, altrimenti **normale**.
- I pezzi speciali possono apparire **contemporaneamente** ai normali (multipli oggetti in scena).

### 4.2 Comportamento in caduta
- Stessa **velocità di discesa** dei pezzi normali (`FALL_SPEED = 3`).
- Se **toccano lo stecchino** (collisione X diff < 28, Y diff < 24):
  1. il pezzo scompare immediatamente (non rimane appeso);
  2. si perde **1 vita**;
  3. viene riprodotto un **suono acuto/gracchiante** (Web Audio, onda a dente di sega + decay rapido);
  4. lo schermo fa un breve **flash rosso** (opacity overlay per 150 ms);
  5. lo schermo fa un **shake** di 5 px per 200 ms (CSS transform animato).
- Se **scende fuori schermo** senza toccare lo stecchino:
  - scompare definitivamente;
  - **NON** ricompare (a differenza dei normali che respawnano finché non infilzati).

### 4.3 Visuali
- Usano i template `#t_pezzo_bad_N` (già presenti in `<defs>`) con cerchi verdi `#00FF00`.
- Devono essere facilmente distinguibili dai normali: la grafica con i puntini verdi è già sufficiente.

### 4.4 Queue vs Caduta Libera
- **Pezzi normali**: rimangono governati dalla coda `pieceQueue` (ordine fisso 1→7) e respawnano se non infilzati.
- **Pezzi speciali**: sono puramente casuali e indipendenti, **non entrano nella coda**.

---

## 5. Collisioni & Ordine di Priorità

Ogni frame vengono valutati tutti gli oggetti in caduta (normale o speciale) nell’ordine naturale dell’array `fallingPieces[]`:
1. Se un pezzo **speciale** collide con lo stecchino → gestione vita (vedi §4.2).
2. Se un pezzo **normale** collide con lo stecchino → gestione infilzamento classico (slide verso `targetY`).

Non è possibile che due pezzi collidano nello stesso frame con lo stesso stecchino perché la hitbox è piccola e il controllo avviene pezzo per pezzo.

---

## 6. Suoni

| Evento | Suono |
|---|---|
| Infilzamento pezzo normale | Oscillatore sine 440→880 Hz, decay 150 ms (esistente) |
| Perdita vita (speciale tocca) | Oscillatore sawtooth 880→200 Hz, decay 300 ms, gain alto, ritardo 0 ms |

Il suono di penalità deve essere nettamente diverso e spiacevole.

---

## 7. Effetti Visivi Penalità

- **Flash rosso**: un `<rect>` fullscreen semi-trasparente rosso (#e30613, opacity 0.4) appare per 150 ms sopra il `game-area`, poi svanisce.
- **Screen shake**: `#Poster` riceve una classe CSS con `transform: translate()` oscillante per 200 ms.

Entrambi sono effetti momentanei, non influenzano il gameplay (la partita continua durante lo shake).

---

## 8. Condizione di Vittoria e Sconfitta

- **Vittoria**: `pieces.length === 7` (tutti i pezzi normali infilzati). Stato `WIN`.
- **Sconfitta**: `lives === 0`. Stato `GAME_OVER`.
- Se perdi l’ultima vita **mentre** hai già 7 pezzi infilzati (teoricamente impossibile, ma per sicurezza): vince la vittoria.

---

## 9. Dati e Costanti Aggiuntivi

```javascript
const BAD_CHANCE_MIN = 0.30;
const BAD_CHANCE_MAX = 0.60;
let currentBadChance = 0; // estratto all’inizio partita
const LIVES_MAX = 3;
let lives = 3;
```

Array `fallingPieces[]` (sostituisce il singolo `currentPiece`) contiene oggetti:
```javascript
{
  type: string,        // es. 't_pezzo_3' o 't_pezzo_bad_5'
  x: number,
  y: number,
  targetY: number|null, // null per speciali
  isBad: boolean
}
```

---

## 10. Compatibilità con Codice Esistente

- I 7 template `t_pezzo_1..7` e i relativi `targetY` restano invariati.
- Il layer `#landed-pieces` continua a ospitare solo pezzi normali infilzati.
- I pezzi speciali non hanno mai `use` in `#landed-pieces`.
- `pieces[]` mantiene il suo significato: solo pezzi normali infilzati.

---

## 11. Reset Completo

La funzione `resetGame()` dovrà:
1. svuotare `fallingPieces[]` e rimuovere tutti i `<use>` temporanei;
2. svuotare `pieces[]` e `#landed-pieces`;
3. resettare `lives = 3` e rendere visibili tutti i cuori;
4. riportare `stecchinoX` al centro;
5. nascondere testi, icone, confetti, flash, ecc.

---

## 12. Riepilogo Logica Spawn

```
all'avvio partita:
  currentBadChance = random(BAD_CHANCE_MIN, BAD_CHANCE_MAX)
  spawnNormalPiece()  // pezzo 1 dalla coda

ogni volta che serve un pezzo:
  if random() < currentBadChance:
    spawnBadPiece()
  else:
    if coda non vuota:
      spawnNormalPiece()
    else:
      // niente più normali da spawnare, eventualmente
      // si può decidere di non spawnare nulla o spawnare solo bad
      niente
```

I pezzi speciali cadono libera; se sfuggiti, scompaiono. I normali che sfuggono ricadono dall’alto finché non infilzati.

---

*Fine del design.*
