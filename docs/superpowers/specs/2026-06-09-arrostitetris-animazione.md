# ArrostiTetris — Animazione Caduta Pezzi

## Obiettivo
Animazione browser (HTML/JS) basata sul poster SVG esistente: i 7 pezzi (`pezzo_1`–`pezzo_7`) cadono uno alla volta dall'alto curvando verso lo stecchino, che segue il mouse. I pezzi si accumulano sullo stecchino formando uno spiedino.

## Stack
- Singolo file `index.html`
- SVG originale inline (pezzi spostati in `<defs>`, istanziati con `<use>`)
- JS vanilla con `requestAnimationFrame`
- Nessuna libreria esterna

## Flusso Animazione
1. All'avvio lo stecchino è nella posizione centrale del canvas
2. Un pezzo alla volta viene creato in cima all'area di gioco, in posizione X casuale (entro l'area bianca) e Y fissata sopra l'area visibile
3. Il pezzo cade a velocità costante verso il basso
4. A ogni frame la X del pezzo viene corretta per curvarlo gradualmente verso la X corrente dello stecchino (attrazione)
5. Quando il pezzo raggiunge la Y dello stecchino + un offset incrementale (per l'accumulo), si ferma
6. Dopo un breve delay parte il pezzo successivo
7. Quando tutti i 7 pezzi sono infilati, il ciclo ricrea in ordine casuale (loop infinito)
8. L'ordine dei pezzi è random a ogni ciclo

## Movimento Stecchino
- Lo stecchino segue il mouse orizzontalmente
- Vincolato entro l'area bianca (x tra ~51 e ~344)
- Tutti i pezzi già infilati si muovono solidali con lo stecchino (stesso gruppo di traslazione)
- La posizione Y dello stecchino è fissa

## Fisica della Caduta
- Velocità verticale costante (px/frame)
- Attrazione laterale: ogni frame la X del pezzo converge verso la X dello stecchino con un fattore proporzionale alla distanza
- Nessuna accelerazione o gravità variabile

## Targeting Y sullo Stecchino
- Il primo pezzo si ferma sulla punta dello stecchino (Y ~530)
- Ogni pezzo successivo si accumula più in alto di ~36px (l'altezza di un pezzo)
- Lo stecchino originale è lungo: la Y dello spiedino cresce da ~530 verso l'alto fino a ~210

## Struttura del File
- `index.html`: contiene il SVG modificato + stili + script

## Modifiche al SVG
- I `<g id="pezzo_N">` vengono spostati dentro `<defs>` e rinominati come template
- Lo stecchino rimane nel DOM ma avvolto in un `<g transform="translate(x,0)">` controllato da JS
- I pezzi vengono istanziati con `<use href="#pezzo_N">` dentro il gruppo dello stecchino per ereditarne la posizione X dopo l'atterraggio
- Sfondo, cuori e testo rimangono invariati

## Casi d'Angolo
- Mouse fuori dalla finestra: lo stecchino resta all'ultima posizione nota
- Ridimensionamento finestra: le coordinate SVG sono assolute, nessun effetto
- Page visibility change: l'animazione si mette in pausa con `document.hidden`
