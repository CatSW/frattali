---
title: Changelog fractal-explorer
copyright: "© 2026 Stefano Vesco (IK0VCK) - CatSW. All rights reserved."
author: IK0VCK
type: changelog
product: fractal-explorer
license: MIT
updated: 2026-11-21
---
# Changelog — Esploratore di Frattali

Formato ispirato a [Keep a Changelog](https://keepachangelog.com/it/1.1.0/), versionamento semantico adattato:
- `0.1.0` e `0.2.0` = i due proof of concept iniziali (pre-pubblicazione);
- `1.0.0` = prima versione pubblicata su GitHub Pages;
- `1.x.0` = incrementi funzionali successivi (una release per ogni blocco di lavoro concluso).

Le date sono nel formato `AAAA-MM-GG`.

## 1.8.0 — 2026-11-22 

### Novità
- Uso da telefono: su schermo stretto (< 700 px) le viste si dispongono in colonna, fino a 3; il pulsante «4» diventa «3». In landscape basso (altezza < 500 px) al massimo 2 viste affiancate e il pulsante «4» si nasconde.
- Stato con troppe viste (avvio, `?s=`, «Applica» JSON, rotazione/ridimensionamento): si chiudono le ultime e l'avviso resta in `#status` (e nell'intestazione per qualche secondo). «Indietro» ripristina le viste chiuse da ridimensionamento.
- Input per evento (`pointerType`), senza modalità globale:
  - un dito: sposta la vista oltre ~10 px; durante il gesto trasla la bitmap già calcolata, il ricalcolo parte al rilascio;
  - due dita: pizzico e pan come prima;
  - tocco breve (< 250 ms): marker sull'overlay della vista, con coordinate e, a calcolo concluso, esito del punto (fuga ≈ N it. oppure non fugge) nell'HUD centrale; un secondo tocco altrove lo sposta, un tocco sul marker lo toglie; il marker è ancorato al piano e segue pan e zoom;
  - doppio tocco: zoom ×2 come prima (il primo tocco non lascia modifiche al marker);
  - pressione prolungata (~450 ms): menu con «Julia da qui» (solo su Mandelbrot, nel punto premuto) e «Rimuovi marker» (se presente).
- «Tocca per scegliere c» e «Zoom area» hanno la precedenza sul tocco; mouse e penna invariati.
- Bersagli tattili ≥ 40 px su mobile; `touch-action: none` anche su overlay e cornici; altezze con `100dvh` (fallback `100vh`).
- Testo di aiuto sui gesti touch visibile solo con puntatore touch.

### Correzioni
- Su touch il pan con un dito non partiva se non dopo la pressione prolungata: ora parte dalla soglia di movimento.

### Limiti noti
- Il marker è uno per vista e si azzera al cambio di famiglia; i suoi dati (esito del punto) compaiono solo per Mandelbrot, Julia, Burning Ship, Tricorno, Phoenix, Magnet I, Esponenziale e Seno, e solo a vista ferma e calcolo concluso. Marker e cursore non usano ancora `evalPoint` né `caps` (previsti dalla 1.9.0).
- Il marker non è nello stato JSON/URL né nella cronologia.
- Con 3 viste su telefono la vista è larga e bassa: i gesti sulle viste non fanno scorrere la pagina, che si scorre dai bordi o dalla barra sotto.
- La ✕ e il badge delle viste sono più grandi su mobile e coprono più area dell'immagine.
- Il testo di aiuto in alto cita ancora «1, 2 o 4» viste anche su schermo stretto, dove sono 1, 2 o 3.

## 1.7.5 — 2026-11-22 

### Novità
- Famiglia **Magnet II** (code 9): mappa razionale di grado 3, modalità parametro (z₀=0) e Julia. DE analitico non presente (de:false, gradiente numerico); testo e preset didattici.
- Famiglia **Esponenziale** (code 10): z → c·e^z, modalità parametro (z₀=0) e Julia. Fuga su Re(z)>50 con protezione overflow. DE analitico abilitato. mu continuo da overshoot su Re per evitare banding. Preset biforcazione parabolica (c=1/e) e bouquet di Cantor (c=0,3).
- Famiglia **Seno** (code 11): z → c·sin(z), modalità parametro (z₀=π/2) e Julia. Fuga su |Im z|>50 con protezione sinh/cosh. DE analitico abilitato. mu continuo da overshoot su |Im|. Preset c=1 e c=1+0,1i.
- Aggiornata sezione «Famiglie aggiunte» e testi didattici coerenti.
- Versione 1.7.5 (22 novembre 2026).

### Correzioni
- Nessuna regressione sulle famiglie preesistenti; il ramo computeBand esteso mantiene il comportamento precedente.

### Limiti noti
- Magnet II: nessun Distance Estimator analitico (derivata della composizione razionale² troppo propensa a errori di implementazione).
- Esponenziale e Seno: DE usa la formula classica basata su |z|/|dz|; per mappe trascendenti è un’approssimazione utile per neon/rilievo ma non è la distanza euclidea esatta.
- Nessun deep-zoom/perturbation; limite FP64 invariato con avviso esistente.
- Preset di partenza scelti per leggibilità didattica; viste ottimali possono richiedere ritocco manuale.

## 1.7.0 — 2026-11-22 

### Novità
- Distance Estimator analitico esteso a Phoenix (fam 4) e Magnet I (fam 8).
- Phoenix: derivata rispetto a z₀ con stato (pdx, pdy) accanto a (pzx, pzy); formula dzₙ₊₁ = 2 zₙ dzₙ + q dzₙ₋₁, dz₀ = 1, dz₋₁ = 0.
- Magnet I: w = (z² + c − 1)/(2z + c − 2), z ← w²; dw/dz e ∂w/∂c esplicitati; Julia (jul=1) e Mandelbrot (jul=0) con catena 2 w · dw.
- de: true impostato nelle voci FAM di Phoenix e Magnet I; normalizzazione DE e uso in Neon allineati a Mandelbrot/Julia (incluso /s e fattore 2).
- Guardie isFinite e soglia |den| per evitare NaN nei buffer de/mu.

### Correzioni
- Nessuna regressione sulle famiglie già supportate (Mandelbrot, Julia, Ship, Tricorn, Newton, Nova, IFS, Buddhabrot).

### Limiti noti
- DE analitico non presente su Nova (come richiesto).
- Oltre ~10¹³ di zoom la precisione a 64 bit resta il limite strutturale (messaggio esistente).

## 1.6.5 — 2026-11-22 

### Novità
- Layout a 1, 2 o 4 viste (griglia 2×2): il pulsante «4» si affianca a «1» e «2». Una nuova vista duplica la vista attiva.
- Pulsante «✕» su ogni vista per chiuderla. È disabilitato quando resta una sola vista. Se si chiude la vista attiva, l'attivazione passa alla successiva (o alla precedente se era l'ultima).
- Reflow: dopo una chiusura le viste rimaste occupano lo spazio disponibile. Con 3 viste, 2 stanno nella riga superiore e 1 a tutta larghezza in quella inferiore.
- Colore, badge e cornici restano legati all'id della vista (V1–V4) e non cambiano con aggiunte, chiusure o reflow.
- Stato JSON e link `?s=`: ogni vista porta il proprio `id` e c'è il campo `active`. Supportate da 1 a 4 viste; gli stati 1.5–1.6 (senza `id`) si aprono come prima.
- «Indietro» ripristina anche aggiunte e chiusure di viste.
- Con 3–4 viste il tetto di pixel della qualità scelta è diviso per il numero di viste (il totale non supera quello di una vista singola). Il messaggio di stato segnala «risoluzione adattata».

### Correzioni
- Reflow con un unico debounce (200 ms) per tutte le viste: si ricalcolano solo quelle il cui canvas ha cambiato dimensione e si ridisegnano le cornici. Aggiungere o chiudere una vista non ricalcola più tutte le altre.
- Fullscreen e ridimensionamento finestra ora attivano il reflow anche senza `ResizeObserver`.
- L'aggiornamento progressivo dell'immagine rallenta in proporzione al costo misurato e al numero di viste in calcolo, così l'interfaccia resta reattiva.

### Limiti noti
- Con 3 viste nessun pulsante di layout risulta attivo (i pulsanti sono 1, 2, 4).
- Riducendo il layout (4→2, 4→1) restano la vista attiva e, per il resto, le prime per id.
- Chiudendo la vista 1 scompaiono dal DOM gli id `vp0`, `cv`, `selrect`, `hudl/m/r`; tornano se la vista 1 viene riaggiunta.
- La regola di pixel agisce solo con schermi grandi o ad alta densità (≥2×) o in fullscreen; a 1400×900 e dpr 1 la risoluzione è invariata.
- Con 3–4 viste lo stato mostra solo l'esito di ciascuna vista.

## 1.6 — 2026-11-21

### Novità
- Cornici proiettate: in ogni vista M si vede il campo visivo delle altre viste N, disegnato su un overlay separato dal canvas (nessun ricalcolo dell'immagine). Il rettangolo matematico di N è proiettato in M con `worldToScreen`; con rotazione diversa da zero diventa un quadrilatero.
- Ogni cornice ha il colore della vista N, contorno doppio (scuro e chiaro, leggibile su sfondi chiari e scuri) e un badge con l'identificativo («V2»).
- Clipping sui bordi della vista (Sutherland–Hodgman): se la cornice esce in parte si disegnano solo i lati visibili (linea aperta a «C») con una freccia sul lato tagliato; se è tutta fuori resta una freccia sul bordo, verso N; se M è interamente dentro N, l'intera vista M ha un bordo tratteggiato col colore di N.
- Aggiornamento in tempo reale durante pan, zoom e ricalcolo, con `requestAnimationFrame`.
- Interruttore «Cornici» nella barra (compare con due viste, attivo all'avvio).
- Funziona con tutte le famiglie, IFS e a densità comprese: le cornici dipendono solo dalle viste, non dal tipo di calcolo.

### Correzioni
- Nessuna.

### Limiti noti
- Rimossi i pulsanti + e − dalle singole viste; restano le scorciatoie da tastiera + e − (sulla vista attiva) e la rotella.
- Una cornice più piccola di circa 10 px è ingrandita a 10 px attorno al suo centro, per restare visibile (non è più in scala).
- Le frecce sui lati tagliati compaiono solo per lati lunghi almeno 28 px (al massimo 4).
- Lo stato dell'interruttore «Cornici» non è salvato in JSON/URL: a ogni avvio è attivo.
- Le cornici non sono incluse nel PNG esportato.
- L'interfaccia non ha ancora controlli di rotazione: il caso rot≠0 è gestito dal codice ed è verificato sulle funzioni di calcolo, non dall'interfaccia.

## 1.5 — 2026-11-21

### Novità
- Astrazione Viewport: ogni vista ha canvas, overlay, vista propria {cx, cy, span, rot}, buffer e cache (pre/full/CDF), generazione dei job e HUD. Lo stato frattale/colore resta unico e condiviso.
- Controllo «Viste: 1 / 2». Due viste affiancate su schermo largo (orientamento orizzontale) o impilate su schermo stretto (verticale).
- Vista attiva con bordo e badge nel colore della vista (4 colori fissi, assegnati per id). Tastiera (+ − R B F P), «Salva PNG», «Julia da qui» e «Tocca per scegliere c» agiscono sulla vista attiva; il pannello controlla tutte le viste.
- Ogni vista ha pan, zoom, gesti e HUD propri; a layout 2 ha anche i pulsanti ⌂ + −. Creando la seconda vista si copia la vista attiva.
- Pool di worker condiviso: i job portano `target` (id vista), i risultati sono instradati alla vista corretta, la cancellazione è per vista.
- Cronologia con snapshot globale {views[], layout}: `B` annulla un'azione, anche il cambio di layout.
- `getState` / `applyState`: `views[]` e `layout` ('1' o '2'); gli stati a una vista si caricano come prima.
- Zoom a finestra con mouse: trascinando con il tasto sinistro sulla vista si disegna la finestra dal punto premuto a quello di rilascio, senza premere «Zoom area». Al rilascio fit del rettangolo (con `pushHist()`, limiti di precisione invariati); sotto 8 px non applica nulla. La pressione lunga resta il pan; touch e pulsante «Zoom area» invariati.

### Correzioni
- Il riavvio dei worker per risultati obsoleti rimette in coda i job validi ancora in corso, invece di perderli (con il pool condiviso avrebbe bloccato la seconda vista).
- Nessun errore se mancano ResizeObserver, setPointerCapture o canvas.toBlob.
- Il testo introduttivo citava «Maiusc + trascina», mai implementato: ora descrive il trascinamento col mouse.

### Limiti noti
- `rot` è presente nel modello della vista ma non viene ancora applicato (sempre 0).
- A layout 2 il calcolo si raddoppia; Buddhabrot e Ultra restano pesanti su telefono.
- Passando da 2 a 1 la seconda vista viene scartata; riaprendola si copia la vista attiva.
- La vista attiva non è salvata in JSON/URL (al caricamento è la vista 1).
- Cornici proiettate, layout a 4 e chiusura singola delle viste: release successive.

## [1.4.0] — 2026-11-21

### Novità
- Motore di calcolo a tile quadrati (64×64) ordinati dal centro verso l’esterno, con campo `target` e numero di generazione per cancellazione immediata della coda.
- Suddivisione Mariani-Silver sui tile (solo famiglie mandel/julia/tricorn, e solo con `inmode === 'flat'`): perimetro prima, riempimento dell’interno se tutto interno, altrimenti ricorsione fino a 4×4.
- Costante `MS_ENABLE` per disabilitare Mariani-Silver e confrontare il risultato.
- Parametri Newton: etichette distinte «Rilassamento Re/Im a» vs «Parametro Re/Im a»; i controlli `a0`/`a1` compaiono solo per `eᶻ − a` e `sin z − a·z`.

### Correzioni
- Visibilità e naming dei parametri di Newton resi coerenti con l’uso effettivo della funzione.
- Scarto affidabile dei risultati obsoleti e ricreazione del pool worker in caso di troppo lavoro stale.

### Limiti noti
- Mariani-Silver non è ancora attivo su Burning Ship, Phoenix, Magnet, Nova, Newton (solo le famiglie classiche a iterazione z²/zᵈ).
- Il riempimento flat dell’interno non si applica quando `inmode` è `tp` o `tx` (trappole).
- Su dispositivi molto deboli la ricorsione MS su tile grandi può essere più lenta del calcolo diretto; usare `MS_ENABLE=false` per confronto.

## 1.3.5 — 2026-11-21

### Novità
- Stato JSON e link `?s=`: nuovo blocco opzionale `newton` con `fn`, `a`, `divergenceRadius`, `convergenceTolerance`, `minDerivativeThreshold`, `divergenceColor`, `singularityColor`. Campo assente → default del codice; valori fuori range → clamp; valori non validi → ignorati.
- `getState()` e l'esportazione JSON includono in `newton` solo soglie e colori diversi dal default.
- Soglie del kernel di Newton (raggio di divergenza, tolleranza, soglia sulla derivata) e colori di divergenza/singolarità resi parametrizzabili. Default invariati: JSON e link precedenti producono la stessa immagine.

### Correzioni
- Nessuna.

### Limiti noti
- Nessun controllo UI per le soglie: si impostano solo via «Condividi» (JSON).
- Gli override si azzerano con preset e cambio famiglia; il cambio di funzione dal menu li mantiene.
- `minDerivativeThreshold` non ha effetto su tan e eᶻ−a.
- Con tolleranze molto alte, nei polinomi la soglia di riconoscimento della radice si allarga (16 × tolleranza al quadrato, minimo 1e-6).
- Nova: il kernel la fa passare dal ramo trascendente (`J.trans` indefinito → 0), come nella 1.3. Non toccato in questa release.

## [1.3.0] — 2026-11-21

### Novità
- Famiglia Newton estesa a funzioni trascendenti: sin z, cos z, tan z, sinh z, cosh z, eᶻ−a (a complesso), sin z − a·z (a regolabile).
- Passi di Newton in forma chiusa dove possibile (z−tan z, z+cot z, z−sin(2z)/2, z−tanh z, z−coth z, z−1+a e^{−z}) con formule stabili anti-overflow.
- Stati di uscita distinti: convergenza (tinta da radice/indice periodico), timeout (nero), divergenza (rosso), singolarità/NaN (magenta). kind 253/254.
- Indice intero k per radici infinite (round(Re z/π) ecc.) mappato su 8 tinte cicliche della palette.
- Preset dedicati e testo didattico su bacini a strisce e radici periodiche.
- UI: «Esporta JSON» (ex Condividi stato) dopo «Nascondi pannello»; icone su Vista iniziale / Indietro / Zoom area.
- Pan solo con tasto destro o pressione lunga (mobile); Zoom area solo rettangolo; cursore crosshair + coordinate live sotto il puntatore; dopo zoom rettangolare la modalità si spegne.

### Correzioni
- Guard-rail espliciti: |Δz|<1e-12, |f'|<1e-14 → singolarità, |z|>1e6 o non-finito → divergenza, maxIt → timeout.
- nroots e colorize gestiscono sia polinomi finiti sia 8 canali ciclici per trascendenti senza collisione di kind.

### Limiti noti
- Formule di tan/cot/tanh/coth usano clamp su |Re|/|Im| ≤20 per stabilità; ai bordi estremi possono apparire artefatti.
- Nessuna derivata analitica aggiuntiva per DE su trascendenti (de resta −1).
- Verifica automatica 100 punti non eseguita in questo ambiente; consigliato test manuale sui preset nuovi.

## [1.2.0] — 2026-11-21

### Novità
- Le palette sono ora interpolate in OKLCH invece che in sRGB: niente più cali di luminanza nelle transizioni tra colori complementari. Interpolazione dell'arco di tinta più breve; i colori acromatici (C < 1e-4) ereditano la tinta dal vicino cromatico più prossimo; gamut clipping morbido (si riduce il chroma finché il colore rientra in sRGB). LUT sempre a 1024 campioni, con `pos` delle palette (es. «Classica») e chiusura ciclica invariati. Le palette cambiano leggermente aspetto.
- Funzioni pure per le coordinate: `screenToWorld`, `worldToScreen`, `fitRect` (più `panView`), con vista `{cx, cy, span, rot}` e `rot` già gestito. Usate da gesti (pan, pizzico, rotellina, doppio tocco), scelta di *c* e «Julia da qui»; comportamento invariato.
- Pulsante «Zoom area» (toggle) e, su desktop, Maiusc+trascinamento: un rettangolo di selezione indica l'area; al rilascio la vista si centra sul rettangolo e lo contiene per intero rispettando le proporzioni del canvas. Rettangoli con un lato sotto ~8 px sono trattati come clic e annullati; Esc annulla la selezione; su touch, con il toggle attivo, la selezione si fa con un dito. L'operazione è annullabile con «Indietro».
- Il toggle «Zoom area» e «Tocca per scegliere c» sono mutuamente esclusivi.
- Con «Zoom area» attivo il trascinamento seleziona invece di spostare la vista.
- Sulla vista restano solo i pulsanti a schermo intero, con nuova icona; lo zoom con `+` / `−` resta disponibile da tastiera (e con pizzico, rotellina, doppio tocco).

### Correzioni
- Gradienti delle palette senza cali di luminanza tra colori complementari (limite noto della 1.1.0).

### Limiti noti
- Profondità di zoom limitata dalla precisione a 64 bit (circa 10¹³); «Zoom area» non scende oltre questa soglia e lo segnala.
- Distance Estimator numerico (non analitico) per tutte le famiglie oltre Mandelbrot e Julia.
- Ricalcolo completo dell'immagine a ogni variazione anche quando i worker lavorano su frame già scartati.
- Campo `rot` nella vista è salvato (sempre 0) ma ignorato dal motore: le funzioni di coordinate lo gestiscono, il rendering no.
- Lunghezza del link `?s=` limitata dal browser (stati molto grandi possono risultare troppo lunghi).
- Con «Zoom area» attivo non è possibile spostare la vista né pizzicare con le dita sul canvas: va disattivato.

## [1.1.0] — 2026-11-21

### Novità
- Pulsante in testata «Condividi» con icona: copia il link `?s=` (stato serializzato in base64url) negli appunti e mostra conferma.
- Pulsante «Condividi stato» nella barra sotto la vista: apre un pannello con textarea, «Copia JSON», «Applica» e «Chiudi».
- Stato serializzabile versionato (`getState` / `applyState`, schema `{v:1,…}`) con vista, parametri di famiglia, iterazioni, qualità e intero blocco colore (inclusa palette `custom`).
- All’apertura, se è presente `?s=` valido, lo stato viene caricato al posto del preset iniziale.
- Clipboard con fallback su `document.execCommand('copy')` e selezione manuale se anche il fallback fallisce.
- Chiusura pannello con Esc o «Chiudi»; messaggi di esito/errore in `#headmsg` e `#status`.

### Limiti noti
- Interpolazione delle palette in sRGB: possibili cali di luminanza nelle transizioni tra colori complementari.
- Profondità di zoom limitata dalla precisione a 64 bit (circa 10¹³).
- Distance Estimator numerico (non analitico) per tutte le famiglie oltre Mandelbrot e Julia.
- Ricalcolo completo dell’immagine a ogni variazione anche quando i worker lavorano su frame già scartati.
- Campo `rot` nella vista è salvato (sempre 0) ma ignorato dal motore.
- Lunghezza del link `?s=` limitata dal browser (stati molto grandi possono risultare troppo lunghi).

## [1.0.0] — 2026-11-20 — Prima pubblicazione

Release minimale: base POC 0.2.0, funzionalità invariate, più intestazione e una correzione.

### Novità
- Intestazione «Esploratore di Frattali» con i pulsanti «Condividi» e «Guida - Ver. 1.0».
- «Guida - Ver. 1.0» apre il Readme su GitHub in una nuova scheda.
- «Condividi» è predisposto in testata: fino alla 1.1 mostra un avviso chiaro («Condivisione non ancora disponibile»).
- Versione 1.0.0 indicata nel codice e nell'intestazione.
- Parametri di avvio: Mandelbrot e Multibrot, «Vista intera» (d = 2, 300 iterazioni, automatiche), stile personalizzato con palette Pavone metallico, modo esterno Sfumato continuo, interno Tinta unita nera, luce 0,85 / 135° / 45°, rilievo 8, lucentezza 40, cangiante 0,15 (prima: Julia «Neuroni» con interno a trappola punto).

### Correzioni
- Il cursore *Cangiante* ora agisce sulla pendenza anche a luce spenta (stili piatti come *Fuoco piatto*, *Nebulosa*, *Classico piatto*); prima non produceva alcun effetto con Intensità = 0. Aggiornata la nota nella scheda «Luce e rilievo».

### Limiti noti
- Nessun salvataggio/condivisione dei parametri (arriva con la 1.1).
- Interpolazione delle palette in sRGB: possibili cali di luminanza nelle transizioni tra colori complementari.
- Profondità di zoom limitata dalla precisione a 64 bit (circa 10¹³).
- Distance Estimator numerico (non analitico) per tutte le famiglie oltre Mandelbrot e Julia.
- Ricalcolo completo dell'immagine a ogni variazione anche quando i worker lavorano su frame già scartati.


## [0.2.0] — 2026-10-24 — Pre-pubblicazione (baseline)

Versione di partenza: file HTML singolo, autonomo, senza dipendenze esterne, funzionante offline. Generata il 24/10/2026 (file `POC-Sonnet-5_5-esploratore_frattali_v0.2.html`).

### Famiglie di frattali (10)
- Mandelbrot e Multibrot (potenza `d` da 2 a 8)
- Julia (con potenza `d` e parametro `c` regolabili)
- Burning Ship e varianti: Celtic, Perpendicolare, Nave perpendicolare, Heart, anche in modalità Julia
- Tricorno e Multicorn
- Newton (6 polinomi, parametro rilassato `a` complesso)
- Nova
- Phoenix
- Magnet I (Mandelbrot e Julia)
- Buddhabrot (traiettorie)
- IFS e chaos game (felce di Barnsley, albero, drago di Heighway, curva di Lévy, triangolo di Sierpiński)

### Navigazione
- Pan e zoom con gesti (mouse e touch) e pulsanti `+` / `−`
- Cronologia con ritorno indietro (fino a 60 passi)
- Preset per ogni famiglia, con vista e stile associati
- «Julia da qui» e «Tocca per scegliere c» per passare dal Mandelbrot al Julia corrispondente
- Schermo intero e pannello nascondibile
- Scorciatoie: `+`, `-`, `R` (vista iniziale), `B` (indietro), `F` (schermo intero), `P` (pannello)
- Iterazioni automatiche in funzione dello zoom; selettore di qualità

### Colore e illuminazione
- 12 palette (inclusa una personalizzabile) e 9 stili pronti
- Modi esterni: liscio, normalizzato (equalizzato), neon (Distance Estimator), stelo
- Modi interni: tinta piatta, trappola d'orbita (punto / asse)
- Densità, scorrimento e animazione dei colori
- Illuminazione con rilievo (Blinn-Phong): azimut, elevazione, rilievo, lucentezza, iridescenza
- Dithering per attenuare il banding

### Motore di calcolo
- Pool di Web Worker con calcolo a bande, avanzamento progressivo e ripiego sul thread principale se i worker non sono disponibili
- Distance Estimator analitico per Mandelbrot e Julia

### Esportazione
- Salvataggio dell'immagine in PNG

### Limiti noti
- Il cursore *Cangiante* non ha effetto con stili a luce spenta (`light = 0`).
- Interpolazione delle palette in sRGB: possibili cali di luminanza nelle transizioni tra colori complementari.
- Profondità di zoom limitata dalla precisione a 64 bit (circa 10¹³); oltre compare l'avviso «limite di precisione raggiunto».
- Distance Estimator numerico (non analitico) per tutte le famiglie oltre Mandelbrot e Julia.
- Ricalcolo completo dell'immagine a ogni variazione anche quando i worker lavorano su frame già scartati.
- Nessun salvataggio/condivisione dei parametri.

## [0.1.0] — 2026-10-24 — Pre-pubblicazione (baseline)

Versione di partenza: file HTML singolo, autonomo, senza dipendenze esterne, funzionante offline. Generata il 24/10/2026 (file `POC-Sesame-esploratore_frattali_v0.1.html`).

### Famiglie di frattali (7)
- Mandelbrot e Multibrot (`d` regolabile)
- Julia (`d` e parametro `c` regolabili)
- Burning Ship (versione base)
- Tricorno e Multicorn
- Newton, Nova e Phoenix
- IFS e chaos game (felce di Barnsley, albero, drago di Heighway, curva C di Lévy, triangolo di Sierpiński)

### Navigazione e colore
- Pan e zoom con gesti e pulsanti `+` / `−`, cronologia indietro, preset, «Julia da qui», «Tocca per scegliere c» e animazione colori
- 12 palette (inclusa personalizzata), 8 stili pronti, modi esterni/interni, illuminazione con rilievo (Blinn-Phong) e salvataggio PNG

### Motore di calcolo
- Pool di Web Worker (fino a 8 worker) con calcolo a bande, anteprima a bassa risoluzione e ripiego sul thread principale

### Limiti noti
- Assenti Magnet I, Buddhabrot e varianti del Burning Ship presenti nella 0.2.0.
- Mancano schermo intero, pannello comprimibile e scorciatoie da tastiera.
- Stessi limiti di precisione a 64 bit (~10¹³), iridescenza legata a `light > 0` e ricalcolo completo su cambi rapidi.

---  

[@IK0VCK]: https://github.com/IK0VCK