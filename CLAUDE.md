# Stecca d'Oro — biliardo all'italiana, goriziana, carambola e boccette

Specialità: all'italiana (5 birilli), goriziana (9 birilli, con varianti classica/tutti doppi/filotto), tre sponde, carambola libera, boccette (a mano). Gioco di biliardo in italiano, tutto in un unico file: `index.html` (HTML, CSS e JavaScript inline, nessuna build, nessuna dipendenza oltre ai Google Fonts). Da pubblicare su GitHub Pages: il file deve chiamarsi `index.html` e stare nella radice del repository.

Il gioco deve funzionare bene su PC, smartphone (verticale e orizzontale) e LIM (lavagna interattiva, schermi da 1920×1080 in su). Ogni modifica va verificata su tutti e tre.

## Regole di lavoro

- **Non toccare ciò che non serve.** Fisica, regole, storia e minigiochi vanno bene così: cambia solo ciò che la richiesta riguarda.
- Interfaccia e testi in italiano.
- **Il tavolo deve restare sempre visibile.** Il pannello dell'effetto (visuale sulla palla e inclinazione della stecca) è facoltativo e sta a lato (in basso sui telefoni); quando è aperto il tavolo si restringe. Non rimetterlo al centro dello schermo: l'utente l'ha bocciato.
- **Il colpo guidato parte spento** e si attiva solo se il giocatore lo sceglie (pulsante «Guida» o menu).
- **Credibilità fisica**: un birillo deve deviare appena la bilia (toglie solo il 5% della velocità verso di sé: pesa pochi grammi). Le bilie devono scorrere a lungo e arrivare piano sul castello: non alzare la resistenza al rotolamento (`mur` ≈ 0,0075 a 30 °C).
- **Rallentatore** solo quando una palla si avvicina al **birillo rosso**, non a tutto il castello.
- **Storia con 3 vite**: ogni sconfitta (o incontro abbandonato tornando al menu) toglie una vita; a zero si ricomincia dal primo circuito. Non rendere la storia ripetibile all'infinito.
- Le regole seguono i regolamenti reali (FIBiS): prima di inventarne, verificarle. Gli aneddoti della «Storia del biliardo» incerti vanno presentati con «si dice / si racconta».
- Dopo ogni modifica l'utente ricarica `index.html` su GitHub: ricordaglielo.

## Struttura del file

La prima riga di `index.html` è uno scheletro (`<!doctype html>`, `<meta charset>`, `<meta name=viewport …>`, stile di reset). Non toglierla: senza il meta viewport i telefoni mostrano la pagina da PC rimpicciolita. Il file termina con `</body></html>`.

Dentro, in ordine:
1. `<title>`, link ai Google Fonts (Abril Fatface, Figtree, DM Mono), `<style>` con i token colore su `:root` (tema unico scuro).
2. HTML: HUD in alto (`.hud`, chip dei giocatori `#pc0`/`#pc1`, `#hmid`), palco con canvas (`#stage`, `#cv`), avvisi (`#toast`), pannello effetto (`#spinPanel` con `#ballCv`, `#sideCv`, `#elev`, `#presets`), barra in basso (`.dock` con i gruppi `#dgAim`, `#dgPow`, `#dgPlace`, `#dgWait`), menu (`#ovMenu`), finestra messaggi (`#ovMsg`).
3. Un unico `<script>` in una IIFE con sezioni marcate da commenti `// ---------- nome ----------`.

### Sezioni del codice (in ordine)
- **Costanti e helper**: unità in metri e secondi. Palla `BR`=0,03075 (61,5 mm), birillo `PR`, passo fisico `DT`=1/400 s, sponda in gomma `CW` e cornice `WD`. Assi: x lungo il tavolo, y in larghezza verso l'alto, z verso l'alto dal panno; origine al centro.
- **TABLES**: 6 tavoli (classico, match, circolo, bar, antico, moderno) con misure `L`×`W` e colori di panno e legno.
- **CTXS** (ambienti): bar, circolo, torneo, mare, baita, gala; ognuno ha `lamp` (colore della luce) e `draw()` che disegna lo sfondo attorno al tavolo.
- **Impostazioni**: `cfg` salvato in `localStorage` `sd-cfg` (modalità, specialità, punti, nomi, bravura CPU, tavolo, ambiente, temperatura, `guide` 0/1/2, suoni, `spinOpen`, `slowmo`, `gv` = versione delle impostazioni del colpo guidato). `params(T)` dà i parametri fisici in funzione della temperatura del tavolo (30 °C = standard, tavolo riscaldato).
- **Fisica**: `integ` (strisciamento → rotolamento, attrito di rotazione), `hitCush` (sponde con elasticità che dipende da velocità e temperatura, effetto laterale, rotazione assorbita), `collide` (urto tra palle con "throw"), `pinHits` (birilli), `step(S)`. `strike(ball, sh)` trasforma il colpo (`ang`, `a` destra/sinistra, `b` alto/basso, `el` inclinazione, `p` potenza) in velocità e rotazione 3D: il massé nasce da solo con la stecca inclinata; `squirt` è la piccola deviazione dovuta all'effetto. Lo stato `S` è clonabile con `cloneS` (lo usano colpo guidato e CPU).
- **GAMES**: le specialità con `kind` (`cue` stecca con birilli, `carom` carambola, `bocce` boccette), nome e punti obiettivo; `GORVAR` le varianti della goriziana. Palle: `w` bianca, `y` gialla, `r` pallino rosso; boccette `R1..R4`, `W1..W4`, pallino blu `P` (colore con `ck(id)`).
- **Regole**: `makePins(game)` (italiana e boccette: croce di 5; goriziana: croce di 9; carambola: nessuno), `pinVal(game, idx, pins, vr)`, `evalShot(game, ev, me, op, pins, vr)` che legge il registro eventi (`{k:'b'|'c'|'p', …}`; le sponde hanno `s`: `T` corta in alto, `B` corta in basso, `L` lunghe) e restituisce punti, punti all'avversario, fallo e palla in mano.
  - Italiana: bianco 2, rosso 4 con altri o 10 da solo, casin 3, carambola 4.
  - Goriziana: esterno 2, interno 8, rosso 10 con altri o 30 da solo, pallino 6. **Tiro di sponda (la propria palla tocca una sponda prima dell'avversaria) = tutto doppio**, anche i punti persi. Varianti: tutti doppi (tutto ×2), filotto (tutti doppi, rosso da solo 80, cinque in fila 80).
  - Falli: 2 punti più il valore dei birilli all'avversario, che ha palla in mano. Se la tua palla abbatte birilli dopo aver colpito l'avversaria, i punti vanno all'avversario, senza palla in mano.
  - Carambola (`evalCarom`): 1 punto se la propria palla tocca le altre due (tre sponde: almeno 3 sponde prima della seconda). Chi fa punto continua.
  - Boccette (`evalBocce`, `accosto`, `bocceRank`), regolamento FIBiS: lancio a mano (`strike` con `hand`, `handSpeed`) dalla zona `HANDZ` lungo la sponda corta in basso; pallino posizionato oltre la metà campo da chi ha vinto l'accosto; bocciata d'acchito che deve colpire il pallino di diretto; tira chi è fuori punto; accosto 2/3/5/8; birilli 2, rosso 4, rosso da solo 8; positivi solo se li abbatte pallino o boccia avversaria dopo una sponda (o di carambola), altrimenti all'avversario; tiro che tocca la sponda corta in alto = doppio; tiri irregolari (birilli colpiti di diretto, palle sotto la metà campo colpite di diretto) annullano la boccia e ripristinano la situazione (`G.snap`).
- **Audio**: sintetizzato con Web Audio (`tone`, `noise`, `sfx.*`), nessun file audio.
- **Storia**: `STORY` = 6 circuiti × 3 avversari (nome, emoji, bravura `lv`, specialità — anche boccette e tre sponde —, punti, battuta). Progresso in `localStorage` `sd-story` (`c`, `o`, `stage` 'match'|'mini', `bonus`, `done`, `lives`, `playing` = incontro in corso, che se abbandonato conta come sconfitta). `LIVES`=3, `newStory()`, `hearts()`.
- **Boccette: flusso**: `startGiocata` (fase `pallino`), `bocceNext` (crea la boccia da lanciare, `G.bat`), `bocceAfter`, `endGiocata`. `G.hand` = lancio a mano (niente pannello effetto, niente stecca: `drawHand`). `meId()` = palla di chi tira.
- **Minigiochi**: `MINIS` (tresponde, bersaglio, castello, masse, sponda, accosto — quest'ultimo a mano, con `hand` e `ball`), ognuno con `setup(lv,L,W,R)`, `judge(ev,S,M)` e, facoltativo, `goal(M)`. Livelli 0–4. Nella storia se ne gioca uno tra un incontro e l'altro: superarlo dà un vantaggio pari al 15% dei punti dell'incontro successivo.
- **Stato e flusso**: `G` (partita corrente), `newMatch`, `newMini`, `startTurn`, `setPhase`, `shoot`, `afterShot`, `endMatch`, `miniJudge`, `miniEnd`, `storyMatch`, `storyMini`. Fasi: `place` (palla in mano), `aim` (mira; il pannello effetto è facoltativo), `power` (barra che sale e scende), `roll`, `settle`, `cpu`, `cpuaim`, `idle`. I tiri si alternano sempre.
- **Palla in mano**: `validSpot`, `findSpot`; si piazza nella metà opposta alla palla dell'altro giocatore.
- **Colpo guidato**: `predict()` esegue la fisica vera su una copia. Livello 1: percorso fino al primo contatto e un tratto dopo. Livello 2: tutte le palle, posizioni finali e birilli che cadranno.
- **Rallentatore e zoom** (`cfg.slowmo`, attivo di default): al tiro `cineWindow(S)` simula una copia e trova quando una palla passa vicino al birillo rosso in piedi (lo colpisce o lo sfiora). In quella finestra `update` porta il tempo a 0,28× (`G.ts`) e la camera a `G.zoom` (fino a 3,2×) attorno a `G.focus`; «⏩ Veloce» lo salta. La camera è in `cam()`: `worldTf(c)` la usa, `worldTf(c,true)` disegna senza zoom (lo fa `buildStatic`); `draw` ingrandisce l'immagine statica con lo stesso zoom.
- **Computer**: `AI_LV` (4 livelli), `cpuBegin` genera i tiri candidati (diretti con vari tagli, potenze ed effetti, più di sponda a 360°), `aiWork(ms)` li simula a pezzi senza bloccare lo schermo, poi sceglie il migliore e aggiunge un errore di mira e di potenza che dipende dal livello. `cpuPlace` sceglie dove mettere la palla quando ha palla in mano.
- **Disegno**: `layout()` (adatta il tavolo allo spazio libero, togliendo quello del pannello effetto se è aperto; in verticale il tavolo ruota di 90° con la sponda corta di partenza, x = −L/2, in basso; `view.rot`), `worldTf`, `toWorld`, `buildStatic` (sfondo, cornice, panno, diamanti, punti), `draw` (birilli, percorsi, palle con puntini che mostrano la rotazione, stecca).
- **Interfaccia**: pannello effetto (`drawSpinUI`, `setHit`, `toggleSpin`), mira (`rotAim`, trascinamento sul tavolo), tastiera, `hud`, `toast`, `card`, menu (`renderMenu`).
- **Storia del biliardo**: testo `HISTORY` aperto dal pulsante del menu `#btnHist` con `card(..., wide)`.
- **Ciclo**: `update(dt, now)` e `frame`. **Avvio**: `start()`.

## Insidie note

- `[hidden]{display:none!important}` è nello stile: nascondere gli elementi con `el.hidden`.
- **Layout responsive**:
  - **compatto** (classe `body.compact`, decisa da `applyCompact()` a ogni ridimensionamento): orizzontale con altezza ≤ 520 px, oppure touch (`pointer:coarse`, quindi telefoni e tablet) con altezza ≤ 900 px, oppure a schermo intero sotto i 900 px. HUD e comandi diventano colonne laterali strette (rispettano le `safe-area` dei telefoni con notch), il pannello effetto diventa una striscia bassa, la **barra della potenza diventa verticale e alta** (`powUI` gestisce i due orientamenti). La LIM (altezza 1080) resta fuori;
  - **schermo intero**: pulsante ⛶ nell'HUD (`toggleFs`: Fullscreen API + blocco in orizzontale dove supportato). Su iPhone l'API non c'è: una finestra spiega «Condividi → Aggiungi alla schermata Home» (meta `apple-mobile-web-app-capable`);
  - durante la potenza c'è anche un **anello colorato attorno alla palla** sul tavolo; in verticale la barra occupa tutta la larghezza;
  - verticale (`orientation:portrait`): il pannello effetto va in basso; telefono (`max-width:620px`): HUD su due righe;
  - LIM (`min-width:1600px` e `min-height:900px`): controlli ingranditi con `zoom:1.3`.
- Quando si apre o chiude il pannello effetto bisogna richiamare `layout()` (lo fa già `setPhase`), altrimenti il tavolo resta coperto.
- **Test su telefono**: provare sempre con la prima riga dello scheletro (meta viewport).
- **Test automatici nel browser**: se la finestra di anteprima è nascosta, `requestAnimationFrame` si ferma. Per simulare partite usare una copia di prova con un aggancio temporaneo (per esempio `window.__T={get G(){return G},update,step,strike,predict,…}` prima di `start();`) e chiamare `update(1/60, now)` in ciclo. Non lasciarlo mai nel file pubblicato e cancellare la copia alla fine. Il computer pensa almeno 700 ms di tempo reale, quindi nei cicli simulati sembra "fermo" finché non passa quel tempo.
- Le impostazioni e il progresso della storia restano nel browser (`localStorage`), non nel file.

## Verifica dopo le modifiche

1. Nessun errore in console.
2. Menu, partita (2 giocatori e contro il computer), pannello effetto, storia e sfide su: PC, telefono verticale (375×812), telefono orizzontale (812×375), LIM (1920×1080).
3. Se si toccano fisica o minigiochi: ogni sfida deve restare risolvibile anche al livello 5 (una ricerca a forza bruta sui tiri deve trovare soluzioni).

Server di prova locale: configurazione `biliardo` in `.claude/launch.json` di questa cartella (porta 8797).
