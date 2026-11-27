---
title: Changelog fractal-explorer
copyright: "© 2026 Stefano Vesco (IK0VCK) - CatSW. All rights reserved."
author: IK0VCK
type: changelog
product: fractal-explorer
license: MIT
updated: 2026-11-26
---
# Changelog — Esploratore di Frattali

Formato ispirato a [Keep a Changelog](https://keepachangelog.com/it/1.1.0/), versionamento semantico adattato:
- `0.1.0` e `0.2.0` = i due proof of concept iniziali (pre-pubblicazione);
- `1.0.0` = prima versione pubblicata su GitHub Pages;
- `1.x.0` = incrementi funzionali successivi (una release per ogni blocco di lavoro concluso).

Le date sono nel formato `AAAA-MM-GG`.

## 2.0.5 - 2026-11-27

- Editor delle tappe

### Novità
- Pannello «Viaggio»: lista delle tappe con aggiungi da qui, duplica, elimina, riordino (trascinamento dalla maniglia ⠿ e pulsanti ▲/▼), rinomina; la selezione porta la vista allo stato della tappa.
- Per la tappa selezionata: durata, pausa, easing, rotta e «Aggiorna da qui».
- Timeline con segnaposto proporzionali all'istante di arrivo, scrub (lo slider esistente, `#trscrub`) e durata totale.
- «Anteprima percorso»: polilinea tratteggiata della traiettoria del centro e segnaposto numerati sull'overlay delle viste (solo overlay, nessun ricalcolo del frattale; coordinate ad alta precisione a zoom profondo).
- Annulla/Ripeti (pila, max 30) per tutte le modifiche all'elenco, incluse «Applica JSON» e «Svuota viaggio».
- Su compact/short il pannello Condividi/Viaggio è ora un bottom sheet / drawer; aprire i Controlli lo chiude.

### Comportamento da conoscere
- «Aggiungi tappa da qui» inserisce dopo la tappa selezionata (in coda se nessuna è selezionata).
- Se un'operazione cambia la prima tappa (elimina, sposta da/verso la prima posizione, aggiorna), la base viene ricalcolata e i diff delle altre tappe rigenerati; gli stati assoluti restano uguali.
- La prima tappa ha sempre durata 0; una tappa spostata fuori dalla prima posizione riceve durata 4 s.

### Schema
Nessuna modifica: viaggio v1 (2.0.0).

## 2.0.0 - 2026-11-27

### Novità
- **Scheda «Viaggio»** nel pannello Condividi (tab *Stato* | *Viaggio*): «Aggiungi tappa da qui», «Svuota viaggio», ▶/⏸, ■, cursore di scrub con tempo, JSON del viaggio (Copia / Applica), «Copia link viaggio».
- **Modello dati** (schema v:1, Appendice A): `base` = stato completo; ogni tappa = `{nome, durata, pausa, easing, rotta, diff, fx}`, con `diff` che contiene solo le differenze rispetto a `base` (`views[0]` = vista pilota, cioè quella con id `base.active`; poi `P`, `fam` e i campi di colore). `videoConfig` e `fx` sono conservati e fanno round-trip, ma non sono ancora usati.
- **Semantica del tempo**: la tappa *i* ≥ 1 è una tratta da *i−1* a *i* della durata `durata`, con easing e rotta della tappa *i*, poi `pausa`. Per la prima tappa `durata` non conta (non ha tratta in ingresso). Durata totale = Σ durata (i ≥ 1) + Σ pausa.
- **Modulo puro `Trip`**: `travelToState(viaggio, t[, opts])` → stato completo, senza accesso a S/VPS/DOM (l'app passa solo `opts.paramKind` per distinguere parametri a opzioni/interi/continui).
- **Interpolazione**: zoom in scala logaritmica; rotte *ottimale* (van Wijk–Nuij 2003, ρ = √2, formule riscritte in forma logaritmica per evitare cancellazioni e overflow), *lineare*, *spline* (Catmull-Rom sul centro, riflessione agli estremi). Easing lineare, inOutCubic, inOutSine. Centri in BigInt a virgola fissa con precisione adattiva: funziona fino a span 1e-250 senza perdere cifre. Numerici dei colori lineari; `incol` e `custom` in RGB lineare; discreti (palette, modi, sorgente fase, livelli, newton, griglia) cambiano a metà tratta; se cambia la famiglia, nessuna interpolazione dei suoi parametri: cambio netto a metà tratta, con i `P` della nuova famiglia presi interi.
- **Player**: tempo reale con salto di fotogrammi (un solo calcolo in volo per volta), qualità dinamica (fattore di risoluzione 0,2–1 tramite `dims()`) che si abbassa se il frame supera ~180 ms e risale sotto ~70 ms; in pausa/stop/fine qualità piena e interfaccia risincronizzata. Il fotogramma precedente resta visibile finché non arriva il successivo (niente lampo nero). Una sola voce in cronologia per riproduzione. Scrub a qualità ridotta e piena al rilascio.
- **`applyState(obj, {silent, fast})`**: `silent` = niente `pushHist`, niente messaggi, `qual` e `auto` dell'utente invariati; `fast` = per i fotogrammi, salta `setFam`/`buildParams`/sincronizzazione UI e ricalcola solo se cambiano vista o parametri di calcolo, altrimenti ricolora soltanto. Senza opzioni il comportamento è identico a prima.
- **Link `?t=`**: JSON compatto, `d.` + base64url del deflate (`CompressionStream`), oppure `j.` + base64url del JSON puro se la compressione non c'è. Se `?t=` e `?s=` sono presenti insieme vince `?t=`. `?s=` invariato.
- Versione 2.0.0 (APP_VERSION, pulsante «Guida - Ver. 2.0.0», data in intestazione).

### Correzioni
- Nessuna in questa release.

### Limiti noti
- Provato solo con jsdom (canvas e worker simulati) e con test numerici in Node; fluidità reale, eventuale sfarfallio con colorazione GPU e adattamento della qualità vanno verificati in un browser vero (checklist).
- Con più viste aperte, se durante il viaggio cambiano parametri di calcolo (P, iterazioni, Newton, stripe) si ricalcolano tutte le viste a ogni fotogramma; se cambia solo la vista pilota, solo quella. Con la «Regia» (1 vista) il problema sparisce.
- Parametri e colori nell'interfaccia non si aggiornano durante la riproduzione, ma alla pausa.
- `chp` (c di Julia ad alta precisione) non viene interpolato: omesso tra le tappe, presente solo ai fotogrammi chiave.
- Le tappe cambiano solo la vista pilota; eventuali differenze sulle altre viste e sul layout sono ignorate.
- `videoConfig` e `fx` non sono ancora usati; `audio.enabled` vale `false` nei viaggi nuovi.

### Test eseguiti
- Round-trip viaggio → JSON → viaggio → JSON identico: 3 viaggi con stati reali (zoom profondo Mandelbrot 4 tappe; Julia con cambio famiglia, spline e inOutSine 3 tappe; Newton 2 tappe), sia via JSON sia via link.
- Link per 10 tappe (zoom fino a 1e-30): JSON compatto 1764 B, `?t=` compresso 678 caratteri (senza compressione circa 2350).
- Percorso ottimale: w(S) = w1, u(S) = d, f monotona e f + g = 1 su 7 configurazioni (fino a span 3 → 1e-250); zoom 3 → 1e-80: il bersaglio resta in vista (distanza/span ≤ 0,07) e al fotogramma 0,9999 è a 1e-18 span.

## 1.17.0 — 2026-11-26

- UI adattiva

### Novità
- Modalità UI unica `html[data-ui]` = `wide` (≥900 px, come 1.16), `compact` (verticale <900 px), `short` (landscape con altezza <500 px, es. Pixel 8 orizzontale); impostata da uno script in `<head>` e riusata da CSS e JS (evento `uimode`).
- `compact`: barra inferiore fissa (Vista iniziale, Indietro, Zoom area, Viste 1/2/4, Controlli) e pannello come bottom sheet non modale (45 dvh) con tab Frattale · Colore · Luce · Altro. Chiusura con ✕, Esc, swipe-down sulla maniglia o azioni che richiedono la vista libera; un tap sulla vista non lo chiude (resta il marker).
- `short`: binario verticale di icone a sinistra e drawer da destra con gli stessi tab.
- Tab «Altro»: Cornici, Griglia, Profilo (+ Cancella, Centra, Grafici), Julia dal marker, Anima, Salva PNG, Esporta JSON e versione. Sono gli stessi nodi DOM, con ID e listener invariati; tornando a `wide` rientrano nella barra nell'ordine originale.
- Intestazione compatta su mobile (titolo 20 px, Condividi solo icona, «Guida» senza versione, kicker e data nascosti).
- Slider a tutta larghezza su mobile (etichetta e valore sopra, cursore sotto).
- Info «i» di famiglia, luce e criteri di arresto aperte su `wide` e chiuse altrove (come intro e legenda); criterio unico `data-ui`, non più `pointer:coarse`.

### Correzioni
- Il landscape del telefono (~915 px) non usa più la colonna desktop da 340 px.

### Limiti noti
- «Julia dal marker» non è ancora contestuale: sta in «Altro» e chiude il foglio al clic.
- Il gruppo Profilo non è ancora collassato e Colore non ha il sottogruppo «Avanzate».
- Il tasto Indietro di Android non chiude il foglio.
- Il foglio è in overlay: copre la parte bassa delle viste impilate finché è aperto.
- Verificato solo con test di sintassi e simulazione DOM, non su dispositivo.

## 1.16.6 — 2026-11-26

### Correzioni
- **Julia e Tricorno/Multicorn neri alla selezione iniziale.** Nell'inizializzazione di `S.P` il letterale aveva due chiavi `d` (`d:2` potenza, poi `d:0.7` degli attrattori): in JavaScript prevale l'ultima, quindi tutte le famiglie partivano con `d = 0,7`. Mandelbrot veniva sistemato da `applyPreset(0)` (`p:{d:2}`); Julia e Tricorno no, perché il cambio famiglia non applica il primo preset. Rimosso il secondo `d` dal letterale comune; `S.P.strange` resta con `d:0.7`.
- **`PARAMDEF`:** rimossa la seconda chiave `d` (sovrascriveva «Potenza d» con «Parametro d» −3…3). Aggiunti `PARAMDEF_STRANGE_D` e `paramDefOf(k, fam)` usati in `buildParams` e `applyState`, così gli attrattori strani conservano etichetta e range corretti senza rompere stati JSON salvati.
- **`evalPointParityTest`:** il confronto sul Distance Estimator era finito dentro un commento (`…per s` + `if(!(…))R.deDiff++` sulla stessa riga), quindi `deDiff` restava sempre 0. Il controllo è di nuovo eseguito (come nei Limiti noti della 1.16.5).

### Limiti noti
- Nessun limite nuovo rispetto alla 1.16.5; il bug del DE nel test di parità non è più nei limiti noti.

## 1.16.5 — 2026-11-26

— Deep zoom, parte B

### Novità
- **Perturbazione per Julia (d = 2).** Il motore della 1.16.0 ora lavora anche sulle viste Julia: il centro (z₀) e `c` sono BigInt a virgola fissa con precisione adattiva (sorgente: stringhe decimali). Due orbite di riferimento in BigInt → `Float64Array` (copia strutturata, niente `SharedArrayBuffer`): **A** = orbita del centro con quel `c`; **B** = orbita di 0 con lo stesso `c`, bersaglio del rebasing (`z = Z_n + δz_n`; quando |z| < |δz| o l'orbita A finisce, `δz ← z` e si prosegue su B). `mu`, `kind` e il buffer `ph` sono popolati come nel percorso FP64; HUD «Precisione: estesa (K cifre)» e orbita indicate anche per Julia.
- **`c` di Julia ad alta precisione.** «Julia dal marker» da una vista profonda del Mandelbrot conserva tutte le cifre del marker (prima `toFixed(10)`). Lo stato JSON e il link `?s=` hanno il campo opzionale `chp: ["re","im"]` (stringhe decimali), accettato solo se coerente con `P.cr`/`P.ci` (entro 1e-9); se assente, `c` è il decimale più corto di `cr`/`ci` (es. `-0.7269` resta esattamente −0,7269). La cronologia (`hist`) lo ripristina.
- **Derivata per il Distance Estimator in perturbazione**, scalata col passo del pixel: Mandelbrot `E' = 2·z·E + s` (equivale a `d(δz) = 2(Z+δz)·d(δz) + 1`); Julia `E' = 2·z·E`, `E₀ = s`. Il DE in pixel è `2|z|·ln|z| / |E|`. Restano le guardie (`isFinite`, soglie, 1e9 = non disponibile; in più, un risultato oltre il range Float32 diventa 1e9). Lo stesso vale per cursore/marker/profilo (`evalPoint`).
- **Iterazioni automatiche in profondità.** Stessa crescita, ma con perturbazione (Mandelbrot e Julia d = 2, span < 1e-10) il tetto sale da 50000 a 100000: a 1e-250 la formula dà ~63000 (Mandelbrot, base 300) e ~84000 (Julia, base 400). L'orbita BigInt di 100000 iterazioni costa ~0,2 s e 1,6 MB (×2 per Julia).
- **Limite di zoom 1e-250** (prima 1e-100). Zoom con rotella/tasti, zoom a finestra, «centra sul profilo», stato JSON/URL e cronologia oltre il limite vengono limitati a 1e-250 con il messaggio «Limite di zoom profondo raggiunto (1e-250)» o, per stato/URL, «superato (richiesto …)».
- Due preset Julia: «Deep zoom: punto fisso del coniglio» a ~1e-60 e al limite 1e-250.

### Correzioni
- `drawBifGuides`: `ctx` era un'assegnazione a una variabile non dichiarata (`ReferenceError` a fine calcolo della biforcazione con le guide attive); ora `const ctx=this.ctx;`.
- Underflow in perturbazione (già presente in 1.16.0, ma nascosto dal limite 1e-100): i test di ciclo e di rebasing usavano quadrati (`dd = ex²+ey²`, `dx²+dy²`) che valgono 0 sotto ~1e-154. Con il centro su un ciclo esatto tutti i pixel risultavano «periodici» e l'immagine usciva interamente interna (verificato a 1e-200 e 1e-250). Ora norma scalata, calcolata solo quando le componenti sono già piccole; rebasing in norma 1 sotto |δ| ~ 1e-140.
- Derivata non scalata: sopra ~1e-160 usciva dal range e il DE diventava 1e9 ovunque; ora è scalata (vedi Novità).
- Conversione orbita BigInt → double: oltre 1000 bit si scartano i bit bassi prima di `Number()` (evita `Infinity` per K ≳ 305 cifre).
- Cache orbite: 10 nei worker (era 8), 16 sul main thread (era 12), perché una vista Julia usa due orbite. Se un'orbita manca, il ripiego FP64 usa il centro in double passato nel job (prima trattava gli offset come coordinate assolute).

### Limiti noti
- **Underflow, caso δc ~ 1e-300.** Il double normale arriva a 2,2e-308: δ ~ 1e-300 è ancora rappresentabile, ma δ² ~ 1e-600 va a 0 (innocuo: trascurabile rispetto a 2Zδ). Verificato solo sul motore dei worker, in Node: nei casi provati (punti fissi repulsivi e c = −2) i conteggi di iterazione coincidono con un calcolo BigInt fino a 1e-315; a 1e-320, con il passo del pixel nel range subnormale, compaiono errori. Nell'app il limite è prima: `Number('1e-K')` vale 0 per K > 323, `Number(BigInt)` esce dal range per K ≳ 305 e il passo `span/H` entra nei subnormali sotto ~1e-308. Per questo 1e-250 (margine di ~55 decadi) è il limite; oltre servirebbe un esponente esteso (floatexp), non implementato.
- Il costo per pixel cresce con le iterazioni: i pixel non iperbolici senza ciclo rilevabile (es. vicino alla punta c = −2) eseguono tutte le `maxIt` (fino a ~63000–84000 a 1e-250); a quella profondità il calcolo può richiedere minuti.
- Julia: il rilevamento dei cicli usa solo «3 ritorni con differenza decrescente»; la clausola relativa a |δz| (Mandelbrot) non è applicabile. Come nel percorso FP64, un'orbita che si avvicina a un ciclo repulsivo restando entro 1e-12 e con distanza decrescente può essere scambiata per interna (non osservato nei test).
- La griglia del piano non viene disegnata a zoom profondo (etichette in double), come in 1.16.0. Profilo sezionale e cornici usano le coordinate in stringa/BigInt e funzionano.
- Il backend WebGL2 colora soltanto: la perturbazione resta sui worker (invariato).
- `evalPointParityTest` (da console): la riga del confronto sul DE è finita dentro un commento («…per sif(!(…))R.deDiff++»), quindi il DE non viene mai confrontato. Preesistente, non toccato in questa release.

## 1.16.0 — 2026-11-26

— Deep zoom, parte A

### Novità
- **Zoom oltre il limite FP64 per la sola famiglia Mandelbrot con d = 2.** Si attiva da solo quando `span / H < 1e-13` (H = altezza in pixel della vista) ed è disattivo altrove; sopra la soglia il comportamento non cambia. Obiettivo di questa release: span fino a `1e-100` (lo zoom si ferma lì con un messaggio).
- **Coordinate ad alta precisione per vista.** Sotto `span < 1e-8` la vista porta, oltre a `cx`, `cy`, `span` in double, il centro in `BigInt` a virgola fissa decimale (`hp = {x, y, K}`), con `K = ceil(−log10(span)) + 28` cifre (minimo 40), ricalcolato a ogni cambio di span. La sorgente è la stringa decimale di `cx`/`cy` dello stato `v:1` (se manca `hp`, `String(cx)`).
- **Pan, rotella, pinch, doppio clic/tap, zoom a finestra, «Zoom ×2/÷2» e «Centra sul profilo»** aggiornano il centro in `BigInt` sommando offset in double scalati sullo span: il punto sotto il cursore resta fermo anche a 1e-100.
- **Stato, URL, JSON, cronologia (B) e preset** salvano e rileggono il centro con tutte le cifre (`views[].cx/cy` lunghi; i preset accettano `hx`/`hy`). Il formato resta `v:1`: un lettore vecchio usa i double arrotondati. Gli estremi del profilo sezionale (`profile.a`/`profile.b`) restano stringhe decimali coerenti con la vista.
- **Orbita di riferimento** `Z_{n+1} = Z_n² + C0` con `C0` = centro della vista, calcolata nel main thread in `BigInt` a virgola fissa binaria (≈ K·3,32 + 16 bit) e arrotondata a double in due `Float64Array(maxIt)`. Si ferma alla fuga (|Z|² > 1e6). Ogni worker la riceve una sola volta per copia strutturata (messaggio `orbit`; niente `SharedArrayBuffer`); registro da 12 orbite lato main, cache FIFO da 8 nel worker, riutilizzo a parità di centro, K e `maxIt`.
- **Perturbazione nei worker** (`computePert`): per ogni pixel `c = C0 + δc`, `δz' = 2·Z_n·δz + δz² + δc` in FP64 con rebasing (`|Z_n + δz| < |δz|` oppure orbita di riferimento finita → `δz ← Z_n + δz`, indice a 0). Produce `mu` liscio, `kind`, `tp`, `tx`, `de` e `ph` (con `z = Z_n + δz`) come il percorso normale: colorazione CPU e GPU (1.15.5) invariate. Le posizioni dei job in questa modalità sono offset da `C0`; Mariani–Silver funziona senza modifiche.
- **DE in perturbazione: analitico**, non gradiente numerico: la ricorrenza di `dz/dc` è semplice su `z = Z + δz`, quindi si usa direttamente (scelta dichiarata rispetto all'alternativa prevista).
- **HUD:** per Mandelbrot d = 2 compare «Precisione: FP64» oppure «Precisione: estesa (K cifre)»; il centro è mostrato con tante cifre quante ne distingue il pixel (+4, oltre 30 cifre con «… (+N cifre)»). La riga di stato indica K e la lunghezza/tempo dell'orbita.
- **Cursore informativo, marker, profilo sezionale ed `evalPoint` sono estesi** (non disattivati): sotto la soglia valutano il punto in perturbazione sull'orbita della vista, con Re/Im a precisione estesa. `evalPoint(fam, params, z0x, z0y, cx, cy, maxIt, nw[, deep])` accetta un nono parametro opzionale `deep = {key, dx, dy}`; senza di esso nulla cambia. Marker ed estremi del profilo restano ancorati al piano a qualsiasi zoom (`ptToScreen`/`ptFromScreen`); cornici proiettate fra viste a zoom diverso usano la differenza fra centri in BigInt.
- **Due preset di zoom profondo** (Mandelbrot): nucleo di un mini-Mandelbrot di periodo 27 (span 2e-30, 58 cifre) e di periodo 68 (span 8e-80, 108 cifre), con `Z_p(c) = 0` calcolato con Newton a 200 cifre.

### Correzioni
- `computeMS`: i quattro quadranti ereditavano `r0/r1` della piastrella e ricalcolavano righe poi scartate (fino a 16× di lavoro sui blocchi 4×4). Ora ricevono le proprie righe: risultato identico, calcolo più rapido per tutte le famiglie con Mariani–Silver.
- Rilevazione dei cicli in perturbazione: tolleranza anche relativa a |δz| e conferma su 3 ritorni consecutivi con differenza decrescente. Senza questo, a zoom profondo ogni orbita che indugia vicino a un ciclo repulsivo (es. z ≈ 2 per c ≈ −2) veniva scambiata per interna.

### Limiti noti
- Solo Mandelbrot con d = 2. Le altre famiglie, d ≠ 2 e le viste sotto la soglia in quei casi restano in FP64 con l'avviso «limite di precisione raggiunto». Senza `BigInt` nel browser tutto resta in FP64, senza errori.
- Rimandati alla 1.16.5: Julia, DE dedicato in perturbazione, casi limite estremi e avvisi finali. Oltre 1e-100 lo zoom è bloccato.
- La griglia del piano di Gauss non viene disegnata in modalità estesa.
- Se il centro della vista esce presto (orbita di riferimento corta) e un pixel sopravvive più a lungo, dopo il rebasing sul valore completo la precisione cala; centrare la vista dentro l'insieme o su un mini-Mandelbrot evita il problema. Sui pixel caotici `mu` può differire da un calcolo esatto di ~1e-3; rispetto al percorso FP64 la differenza può essere di più iterazioni, perché il percorso FP64 perde il termine piccolo.
- L'orbita si calcola nel main thread: pochi ms per 10000 iterazioni a K ≈ 110; a `maxIt` molto alti il costo cresce linearmente.
- Preesistente, non toccato: `drawBifGuides` usa `ctx` non dichiarata (ReferenceError al termine del calcolo della biforcazione).

## 1.15.5 — 2026-11-25

### Novità
- WebGL2 colorazione, parte B: la GPU colora ora tutte le famiglie a bande con qualunque stato di colore. Aggiunti nello shader il modo esterno Equalizzato (CDF come texture R32F 2048×1, ricaricata solo quando la CDF cambia) e Steli, gli interni Trappola punto e Trappola croce, il rilievo con luce Blinn-Phong (altezza dai 4 vicini, normali, componente diffusa e speculare), la cangiante (attiva anche a luce spenta) e il dithering Interleaved Gradient Noise.
- Fonti colore di fase sulla GPU: Fase finale (continuo e binario), Stripe average e Fase media, con miscela di peso α in OKLab letta dalla texture `ph`.
- `gpuParityTest()` esteso: matrice completa modo esterno × interno × 9 stili, più ogni fonte di fase × α × due stili. Restituisce righe e riepiloghi per modo, interno, stile e fonte, e li stampa in console. Opzioni: `modes`, `inmodes`, `styles`, `srcs`, `alphas`, `phStyles`, `tol`, `pxTol`, `righe`.
- `?gpu=0` nell'URL forza ancora il percorso CPU per confronto.

### Correzioni
- La LUT della GPU è ora RGBA16F (ripiego automatico su RGBA8): con la luce speculare la quantizzazione a 8 bit della LUT superava la tolleranza di parità.

### Limiti noti
- La parità GPU/CPU è entro ±2/255 su quasi tutti i pixel, non su tutti: pixel isolati (indice di palette che differisce di 1 per float32 contro double) arrivano a 4–12/255. Nella vista iniziale di Mandelbrot sono al più 6 pixel su 364.000 per combinazione.
- Esponenziale e Seno con modo Equalizzato e luce: lo 0,07 % e lo 0,59 % dei pixel supera ±2/255 (max 17/255), nei plateau di μ discreto dove la CPU in double risolve differenze sotto la precisione float32 della GPU. Effetto visivo trascurabile; non corretto in questa release.
- Restano sulla CPU le famiglie a densità (IFS, Buddhabrot, attrattori strani, biforcazione, geometrici) e l'anteprima a bassa risoluzione (1/16 dei pixel) mostrata durante il calcolo.
- Le verifiche di parità sono state eseguite in Chromium headless con WebGL2 software (SwiftShader), non su una GPU fisica.

## 1.15.0 — 2026-11-24

### Novità
- **Colorazione WebGL2 (parte A).** Un canvas WebGL2 (`.vgl`, stesse dimensioni del canvas di vista, `pointer-events: none`, contesto con `preserveDrawingBuffer: true`) si sovrappone al canvas di vista e colora i buffer già calcolati dai worker. Il calcolo non cambia.
- Texture `R32F` per `mu`, `de`, `tp`, `tx`, `ph` e `R8UI` per `kind`, aggiornate tile per tile con `texSubImage2D`; LUT `1024×1` `RGBA8` (`LINEAR`, ripetizione sull'asse della palette). Dati letti con `texelFetch` (campionamento `NEAREST`).
- Fragment shader GLSL ES 3.00 per esterno **liscio** e **neon**, interno **tinta unita**, e stati di uscita di Newton: convergenza per radice (`kind` 2…), timeout (`kind` 1), singolarità (253), divergenza (254), con i colori e le soglie del blocco `newton`. Uniform collegati a Densità, Fase, Spessore, colore interno e colori di divergenza/singolarità.
- `recolor()` e il ciclo di animazione (`animLoop`) sul percorso GPU aggiornano solo gli uniform ed eseguono la draw call: nessuna ricolorazione CPU, nessun ricalcolo.
- Un contesto WebGL2 per vista (fino a 4): `GpuColor` è istanziato da ogni `Viewport` e liberato alla chiusura della vista.
- HUD: «Colore: GPU» / «Colore: CPU» accanto allo zoom di ogni vista.
- Debug: `gpuParityTest()` da console (vista attiva; `gpuParityTest(1)` per la vista 2) confronta GPU e CPU, in `smooth` e `neon`, con tolleranza ±2/255 per canale. `?gpu=0` nell'URL forza il percorso CPU.

### Correzioni
- «Salva PNG» e lo snapshot dei gesti (pan, zoom con rotella) usano l'immagine visibile (anteprima 2D + strato GPU): identici su backend GPU e CPU.

### Limiti noti
- La GPU colora solo con: famiglia a bande, modo esterno `smooth` o `neon`, interno `flat`, **luce 0**, **cangiante 0**, fonte colore «Iterazioni». Altrimenti ripiega sulla CPU senza errori (`equal`, `stalk`, interni `tp`/`tx`, luce/rilievo, iridescenza, fonti di fase `final`/`stripe`/`mean` e peso α). Gli stili pronti con luce > 0 (compreso quello iniziale) restano quindi su CPU fino alla parte B.
- Famiglie a densità (IFS, Buddhabrot, attrattori strani, biforcazione, geometrici): restano su CPU; il worker accumula conteggi in `mu` e colore medio in `de`, e `colorize()` ne ricava luminosità `(log1p(c)/log1p(max))^0,45` e tinta `off + dens·colore medio`.
- Parità: differenza massima 2/255 su tutte le famiglie e su tutte le funzioni Newton provate; su Phoenix e su una funzione Newton 1 pixel (circa 0,001 %) arriva a 3–4/255 per un indice di LUT diverso di una voce (float32 GPU contro double CPU). La GPU campiona la LUT al centro del texel per replicare l'indice discreto della CPU; non replica il dithering ±0,5 della CPU.
- L'anteprima a bassa risoluzione durante il calcolo è ancora colorata dalla CPU.
- Memoria: circa 21 byte per pixel di texture (6 texture), anche se `tp`, `tx`, `ph` non sono ancora letti dallo shader; `GPU_TEX_ALL=false` le esclude.
- Con contesto WebGL perso la vista resta su CPU fino a un nuovo caricamento o a una nuova vista.

## 1.14.9 2026-11-24

### Correzioni

- Famiglia Nova ripristinata (rotta diverse versioni fa)

## 1.14.8 2026-11-24

### Novità
- Nuova famiglia **Geometrici (Koch, albero pitagorico)** con selettore «Figura»: Curva di Koch, Fiocco di Koch e Albero pitagorico (Sierpiński, Heighway e Lévy restano negli IFS).
- Le figure sono generate per ricorsione e rasterizzate sullo stesso buffer di densità di IFS e attrattori: palette, luce e rilievo, qualità ed esportazione PNG funzionano come nelle altre famiglie. Antialiasing analitico (segmenti campionati ogni 0,5 px con splat bilineare, quadrati con copertura esatta); la qualità Alta/Ultra aggiunge il supercampionamento già esistente.
- Slider «Iterazioni» da 1 a 12 e slider «Angolo θ» (5°–85°, visibile solo per l'albero pitagorico); anteprima viva durante il trascinamento.
- Taglio della ricorsione: segmenti e quadrati sotto 0,5 px sullo schermo non vengono suddivisi; l'HUD segnala il taglio e il livello effettivamente tracciato.
- Culling spaziale: ogni sottoalbero fuori dalla vista corrente non viene visitato; pan e zoom ricalcolano solo ciò che si vede.
- Vista iniziale adattata al riquadro della figura e al formato della vista (come per IFS e attrattori); «R» la ripristina. 6 preset (Koch, fiocco, zoom sul vertice a livello 12, albero a 45°, 30° e 60°).
- `caps` come per le famiglie a densità: `escape`, `phase`, `point`, `profile`, `juliaPick` falsi, `grid: 'xy'`; nuova chiave `geo` (solo per questa famiglia) per i controlli propri. I controlli non pertinenti si nascondono con `applyCaps()`.
- HUD: livello n; Curva di Koch: lati 4ⁿ, lunghezza (4/3)ⁿ·L, D = ln 4 / ln 3 ≈ 1,2619; Fiocco di Koch: lati 3·4ⁿ, perimetro (4/3)ⁿ·L, area Aₙ = A₀·(8/5 − 3/5·(4/9)ⁿ) con limite 8/5·A₀, D; Albero pitagorico: quadrati 2ⁿ⁺¹ − 1 e θ, senza alcuna dimensione frattale (il testo didattico spiega perché non è definita come per Koch).
- Triangolo di Sierpiński (IFS): la nota della famiglia riporta D = ln 3 / ln 2 ≈ 1,585.
- Stato JSON (`v:1`): famiglia `geo` con `P: {fig, ang, niter}`; nessun campo nuovo a livello superiore.

### Correzioni
- Nessuna.

### Limiti noti
- Le linee di Koch hanno spessore fisso di circa 1 px.
- Albero pitagorico: n conta le generazioni sopra il tronco (n = 1 → 3 quadrati); dove i quadrati si sovrappongono la copertura è limitata a 1, quindi la densità non cresce.
- Con il taglio attivo il tracciato si ferma al livello indicato nell'HUD; i valori di lati e lunghezza restano quelli teorici del livello n.
- Cambiando figura la vista viene adattata; cambiando angolo o livello resta quella corrente (con angoli piccoli l'albero può uscire dalla vista: «R»).
- Gli stati con `fam: "geo"` non vengono letti dalle release precedenti (famiglia sconosciuta).
- Preesistente, non toccato: `applyState` per gli attrattori strani rifà il fit della vista e ignora la vista salvata nello stato.

## 1.14.7 — 2026-11-24

### Novità
- **Marker con il mouse.** Un clic sinistro singolo senza trascinamento (come il tap su touch) piazza o sposta il marker sul punto cliccato; se il clic cade entro 24 px dal marker presente, lo rimuove. Un secondo clic entro 350 ms (e 32 px) attiva lo zoom ×2 e ripristina lo stato precedente del marker, quindi il doppio clic non lascia marker indesiderati. Il trascinamento col tasto sinistro continua ad aprire la finestra di zoom; il pan resta su tasto destro/centrale, Spazio + trascinamento, pressione prolungata senza movimento.
- **Pulsante unico «Julia dal marker»** (ID DOM `pick` conservato) al posto di «Julia da qui» e «Tocca per scegliere c». Con un marker sulla vista attiva apre subito il Julia con c = marker. Senza marker il pulsante si accende (etichetta «Scegli il punto sulla vista…», messaggio di stato, `aria-pressed`) e il prossimo clic o tocco sulla vista fissa c; premuto di nuovo annulla l'attesa.
- Testi di intestazione e guida (scheda A) aggiornati a marker con clic e a «Julia dal marker».

### Correzioni
- L'attesa del punto per il Julia si azzera con «Zoom area», con la modalità Profilo (e entrando nell'attesa si esce da Profilo, perché lì il clic traccia il segmento), con il cambio di famiglia, con `applyState` e con «Indietro» verso un'altra famiglia. Prima poteva restare accesa e nascosta, e riapparire tornando al Mandelbrot.
- «Julia da qui» usava il centro della vista: ora c è sempre il marker. La voce «Julia da qui» del menu di pressione prolungata su touch resta invariata.

### Limiti noti
- Con il mouse sopra la vista il cursore informativo (hover) sovrascrive la riga del marker nell'HUD; il testo del marker ricompare all'uscita del puntatore.
- Il marker è per vista: il pulsante legge solo quello della vista attiva.
- In «Zoom area» esplicito e in modalità Profilo il clic singolo non piazza marker (le modalità hanno la precedenza). La penna resta invariata.
- Come su touch, il marker col clic vale per tutte le famiglie, anche quelle senza cursore informativo.

## 1.14.5 — 2026-11-24 

### Novità
- Guida didattica, scheda **D · Laboratorio**: cinque esperimenti «Prova tu», ciascuno con consegna (3 righe), pulsanti e domanda di riflessione: 1 «Perché Newton a volte fallisce?» (z³ − 2z + 2 con a = 1 e con a = 1 + 0,5i), 2 «Dentro o fuori il Mandelbrot» (Julia per c in cardioide, bulbo di periodo 3 / coniglio di Douady, esterno), 3 «La strada verso il caos» (cascata dei raddoppi, zoom sui raddoppi, finestra di periodo 3), 4 «Olomorfo vs non olomorfo» (Mandelbrot, Burning Ship, Tricorno in sequenza), 5 «Leggere il profilo» (Mandelbrot con fase finale e segmento A→B preimpostato).
- Gli stati sono costanti JSON (schema `v:1`) nell'oggetto `LAB`, facilmente modificabile; ogni pulsante li applica con `applyState` (con cronologia: B torna indietro) e scorre con scroll fluido fino alla vista attiva (senza animazione se il sistema richiede meno movimento).
- Pulsanti con bersaglio tattile ≥ 44 px, testo a capo, nessun overflow orizzontale su telefono.
- Etichetta «Guida - Ver. 1.14.5» e `APP_VERSION` / `APP_VERSION_SHORT` aggiornate.

### Correzioni
- Nessuna.

### Limiti noti
- Le viste affiancate mostrano una sola famiglia alla volta: l'esperimento 4 applica i tre stati uno dopo l'altro, non in tre viste contemporanee.
- Ogni pulsante sostituisce le viste correnti con una sola e riporta palette, modo e tinta interna a valori fissi (nero); le altre regolazioni (luce, rilievo, qualità) restano quelle correnti.
- Il «coniglio di Douady» (c = −0,123 + 0,745i) è nel bulbo di periodo 3, non sul bordo del Mandelbrot.
- Se `applyState` o lo scroll non sono disponibili nel browser, i pulsanti non fanno nulla o non scorrono, senza errori.
- Esistono due chiavi `d` duplicate (in `S.P` e in `PARAMDEF`): la seconda prevale sulla prima. Non toccato in questa release.

## 1.14.0 — 2026-11-23

### Novità
- Guida in-app, scheda **B «Storia e riferimenti»**: dodici mini-schede (chi, quando, cosa) su Fatou e Julia, Mandelbrot, Newton e Cayley, Burning Ship, Tricorno/Mandelbar, Phoenix, Magnet, Hénon/Clifford/de Jong, mappa logistica e Feigenbaum, Koch, IFS e Barnsley, Buddhabrot. Ogni scheda ha almeno un link di approfondimento (`target="_blank" rel="noopener"`).
- Layout compatto per telefono: titoli e testo a larghezza piena, link con area di tocco di almeno 40 px.
- Versione aggiornata a 1.14.0 (`APP_VERSION`, `APP_VERSION_SHORT`, pulsante «Guida - Ver. 1.14.0», riga «Versione»).

### Correzioni
- Nessuna in questa release. Schede A e C invariate; ID DOM, scorciatoie, preset, cronologia e funzioni pubbliche non toccati.

### Limiti noti
- I link esterni richiedono connessione; offline l'app funziona normalmente, i link non si aprono.
- I link sono verso Wikipedia in inglese, o verso pagine di Paul Bourke dove non è stata trovata una voce Wikipedia adatta (Phoenix, Magnet, Clifford, de Jong). Non sono state usate pagine in italiano.
- La scheda Koch non rimanda a un controllo dell'app: la famiglia Koch non è ancora presente.
- La scheda **D «Laboratorio»** resta un segnaposto per la release successiva.

## 1.13.5 — 2026-11-23

### Novità
- Guida didattica a fisarmonica (una scheda aperta alla volta) al posto delle sette voci «Come nascono i colori»: **A Manuale breve**, **B Storia e riferimenti**, **C Applicazioni**, **D Laboratorio**.
- **A Manuale breve**: navigazione (mouse, touch e tastiera), «Julia da qui» e «Tocca per scegliere c» con la dicotomia di Fatou–Julia, iterazioni e limite di precisione, perché il nero (con il caso Mandelbrot/Julia e il caso Newton), colori, legenda e rilievo, «Fonte colore», cursore informativo, griglia (G) e Profilo; rimando al manuale completo (README su GitHub).
- **C Applicazioni**: analisi numerica, sistemi dinamici, fisica statistica, antenne frattali, compressione e grafica procedurale, biologia e Terra.
- **B** e **D** sono presenti nella struttura come segnaposto («In preparazione»).
- `makeInfo` accetta `opts.title`/`opts.letter`/`opts.sub` (intestazione di fisarmonica, nome accessibile dal testo visibile) e `opts.noEsc` (il pannello non si chiude con Esc). Le didascalie «i» esistenti non cambiano.
- Pulsante «Guida - Ver. 1.13.5», `APP_VERSION` e `APP_VERSION_SHORT` aggiornati.

### Correzioni
- Nessuna.

### Limiti noti
- Le schede B e D sono segnaposto: contenuti previsti nelle release successive (1.14.0 e 1.14.5).
- Le sette voci rimosse (Sfumato continuo, Luce e rilievo, Neon e steli, Fase dell'orbita, Interno ed equalizzato, Famiglie aggiunte, Limiti) non sono più in pagina. Restano la descrizione di ogni famiglia («i»), la legenda e il testo sui criteri di arresto; le formule delle «Famiglie aggiunte» e i dettagli su neon/steli e interno equalizzato non sono più riassunti da nessuna parte e andrebbero ricollocati in B/D o nel manuale.
- Il testo di A cita 10¹³ ingrandimenti come ordine di grandezza del limite di precisione; la soglia effettiva dipende dalla posizione (stato: «limite di precisione raggiunto»).

## 1.13.0 — 2026-11-23

### Novità
- Nuova famiglia **Biforcazione (mappa logistica)**: diagramma di biforcazione a colonne deterministico della mappa \(x \mapsto r\,x(1-x)\), integrato nel percorso densità/istogramma (buffer `cnt`/`col` come IFS/Buddha/attrattori).
- Selettore asse **r** (intervallo tipico [2,5 – 4]) / **c** (coniugazione \(c = \frac{r}{2}(1-\frac{r}{2})\), ramo \(r = 1+\sqrt{1-4c}\)).
- \(N_{\mathrm{trans}}\) e \(N_{\mathrm{plot}}\) adattivi allo zoom e alla qualità; pan/zoom su entrambi gli assi; griglia `rx`.
- Linee guida verticali opzionali (r₁=3, r₂≈3,449, r_∞≈3,570, finestra periodo 3 ≈3,828), convertite su c quando serve.
- Preset: vista intera (r/c), cascata 3,4–3,6, finestra periodo 3.
- HUD: valore di r o c e di x sotto il cursore; testo didattico con legame r↔c e richiamo a Feigenbaum δ≈4,6692.
- `caps`: escape/phase/point/profile/juliaPick false, grid `'rx'`; controlli non pertinenti nascosti via `applyCaps`.

### Correzioni
- Nessuna (release additiva).

### Limiti noti
- Precisione a 64 bit: zoom molto profondi sulle finestre periodiche possono mostrare artefatti numerici sulla mappa logistica.
- Il motore è a colonne intere (un job deterministico per vista); su schermi molto larghi in qualità Ultra il calcolo può richiedere qualche secondo.
- Le linee guida usano valori di riferimento fissi (non calcolati dinamicamente nell’app).

## 1.12.0 — 2026-11-23

### Novità
- Nuova famiglia **Attrattori strani** (motore a densità, stesso schema di accumulo di IFS/Buddhabrot):
  - varianti Clifford, de Jong, Hénon con parametri a, b, c, d (c e d nascosti per Hénon);
  - scarto del transitorio (~100 iterazioni) e ripartenza su divergenza numerica;
  - inquadratura iniziale da bounding box empirico (`fitStrange`);
  - numero di campioni scalato con qualità e zoom;
  - preset rappresentativi per le tre varianti;
  - `caps`: escape/phase/point/profile/juliaPick falsi, griglia `xy`;
  - stima dell’esponente di Lyapunov massimo (Benettin) nell’HUD quando disponibile;
  - nota didattica in testo famiglia e sezione «Famiglie aggiunte».

### Correzioni
- Nessuna: i motori a bande e le altre famiglie non sono stati modificati nel comportamento.

### Limiti noti
- La stima di λ è una media grezza su batch di campioni; a zoom molto spinti o parametri al limite della stabilità può oscillare o risultare poco significativa.
- Come per IFS/Buddhabrot, a qualità Ultra e molti campioni il calcolo può richiedere alcuni secondi su dispositivi mobili.

## 1.11.0 — 2026-11-23

### Novità
- **1.11.0 — Profilo B: pannello grafici 75/25.** Sotto le viste (desktop) o come bottom sheet (~40% altezza, mobile) compare un pannello dedicato ai grafici del segmento profilo sezionale (1.10.5). Non è una Viewport: non conta nel limite di viste e non usa buffer di calcolo frattale.
- Due canvas 2D ridimensionati con `ResizeObserver`: superiore 75% (grandezza selezionabile), inferiore 25% (fase `arg zₙ`), stesso asse `t ∈ [0,1]` con estremi A/B e distanza `|z_B−z_A|`.
- Campionamento `N = min(1024, larghezza px del pannello)` con `evalPlane`/`evalPoint` a blocchi via `requestAnimationFrame`; durante il trascinamento degli handle throttle ~30 ms e annullamento del job precedente.
- Grandezza superiore (menu): **Iterazioni continue μ** (default), **Potenziale log|zₙ|/dⁿ**, **log₁₀|z_N|**, **Stima di distanza** (campo `de` di `evalPoint`). Traccia colorata con la stessa LUT/mapping di base della vista (offset, densità, colore interno).
- Grafico fase: linea in (−π, π] interrotta ai salti > π (niente segmenti verticali spuri).
- Transizioni interno↔esterno: bisezione (~12 passi) sul cambio di `kind`; linee tratteggiate gialle su entrambi i grafici e segni sul segmento nella vista, con etichette «ingresso nella regione che non diverge» / «uscita».
- Hover/tocco sui grafici: cursore sul segmento nella vista e lettura numerica in `pmeta`.
- Pulsante **Grafici** (accanto a Centra sul segmento) e **Copia CSV** (`t,Re,Im,mu,arg,kind,n,mod,de`) con lo stesso fallback clipboard di Condividi. Apertura automatica del pannello al primo segmento tracciato; chiusura alla cancellazione del profilo.

### Correzioni
- Nessuna regressione intenzionale sul tracciamento profilo 1.10.5 (handle, pan con Spazio/tasto centrale/due dita, serializzazione JSON `profile`).

### Limiti noti
- Il potenziale e la DE sono definiti in modo coerente solo per le famiglie con fuga e, per la DE, dove `evalPoint` fornisce `de ≥ 0` (Mandelbrot/Julia/Phoenix/Magnet I/exp/sin). Altrimenti i campioni risultano vuoti su quel canale.
- Su dispositivi lenti, con N alto e molte transizioni, il campionamento resta asincrono ma può richiedere qualche frame; il pan della vista non è bloccato.
- Il pannello grafici non è incluso nello stato JSON/URL (solo il segmento profilo lo è, come in 1.10.5).

## 1.10.5 — 2026-11-23

### Novità
- Modalità **Profilo** (scorciatoia **O**, pulsante in barra) per le famiglie con `caps.profile`: trascinamento nella vista attiva traccia il segmento A→B sul piano complesso.
- Overlay del segmento: linea tratteggiata viola con contorno doppio, handle circolari etichettati «A» e «B»; il segmento è memorizzato in coordinate mondo e resta ancorato a pan/zoom.
- Trascinamento degli handle (area di presa ≥14 px mouse / ≥28 px touch) con feedback cursore; pan in modalità Profilo tramite due dita, Spazio+trascinamento o tasto centrale.
- Pulsanti **Cancella profilo** e **Centra sul segmento**; Esc esce dalla modalità lasciando il segmento.
- Blocco opzionale `profile` in stato JSON/URL (`view`, `a`, `b`); il profilo si rimuove se la vista associata viene chiusa.

### Correzioni
- Nessuna.

### Limiti noti
- Un solo profilo alla volta (associato a una vista); non c’è ancora il pannello grafici (previsto in 1.11.0).
- In modalità Profilo il pan a un dito è disattivato di proposito (serve a tracciare).


## 1.10.0 — 2026-11-23

### Novità
- **Modi di fase basati sull'argomento dell'orbita** per le famiglie con `caps.phase === true` (Mandelbrot e Multibrot, Julia, Burning Ship e varianti, Tricorno e Multicorn, Phoenix).
- **Buffer `ph`** (`Float32Array`) per pixel, trasportato come `mu`, `de`, `kind`: calcolato nei worker, restituito da bande e tile, copiato nei buffer di vista, unito nel percorso Mariani-Silver. Convenzione: `ph` vale 0 per i punti interni e con fonte «Iterazioni»; per gli esterni è `θ/2π` in [0, 1) (fase finale, fase media) oppure la media delle strisce in [0, 1] (stripe average).
- **Selettore «Fonte colore»**: «Iterazioni» (predefinito, rendering identico alla 1.9.5), «Fase finale», «Stripe average», «Fase media (sperimentale)». Si applica solo all'esterno; l'interno resta governato da «Interno». Il selettore e i sotto-controlli sono nascosti tramite `data-cap="phase"` nelle famiglie senza fase.
- **Fase finale**: θ = atan2(Im zₙ, Re zₙ) all'istante di fuga, in [0, 2π), come posizione ciclica nella palette. Opzione «Livelli»: continuo oppure 2 (segno di Im zₙ).
- **Stripe average** (Härkönen): media di 0,5 + 0,5·sin(s·θₖ) lungo l'orbita, saltando z₁ e z₂; s intero da 1 a 32 (predefinito 5). Interpolazione: `ph = f·Aₙ + (1−f)·Aₙ₋₁`, con `f = 1 − log_d(ln|zₙ| / ln R)`, cioè la parte frazionaria dell'iterazione liscia.
- **Fase media** (sperimentale): media circolare `atan2(Σ sin θₖ, Σ cos θₖ)` con gli stessi k della stripe average.
- **Peso α** [0, 1] (predefinito 1): 0 = solo iterazioni, 1 = solo fonte di fase; i valori intermedi mescolano due posizioni della stessa palette nello spazio OKLab. Visibile solo con fonte diversa da «Iterazioni». «Livelli» e α ricolorano senza ricalcolare.
- **Raggio di fuga**: invariato, R = 1000 per ogni fonte (già ≥ 100); `mu` non cambia.
- **Legenda e cursore**: con una fonte di fase l'asse mostra 0–2π (π/2, π, 3π/2) oppure [0, 1] per la stripe average, e l'indicatore segue il valore di fase. Il cursore e il marker touch aggiungono una riga con il valore della fonte. `evalPoint` restituisce anche `ph`.
- **Stato JSON/URL**: campi opzionali `src` (`final`, `stripe`, `mean`), `alpha`, `stripeS`, `levels` (`2`), omessi se coincidono con i valori predefiniti e ignorati per le famiglie senza fase.
- **Preset didattici** con descrizione di 2 righe: Mandelbrot «Fase finale: raggi esterni», «Fase binaria (decomposizione binaria)», «Stripe average: valle dei cavallucci marini»; Julia «Stripe average (neuroni)». I preset preesistenti non toccano la fonte colore.
- Nuova voce «Fase dell'orbita» nella guida in pagina.

### Correzioni
- Nessuna.

### Limiti noti
- Il futuro backend GPU (WebGL2) e la modalità a perturbazione/deep zoom dovranno leggere e scrivere il buffer `ph` con la stessa convenzione, altrimenti le fonti di fase non saranno disponibili o daranno risultati diversi.
- La fase finale mostra salti a gusci tra punti che escono a iterazioni diverse: è propria del metodo e non viene interpolata. La fase media non è interpolata e può mostrare rumore.
- Il rilievo (luce) resta calcolato dal campo μ, non dalla fonte di fase.
- Le fonti angolari usano cicli interi di palette (Densità arrotondata, minimo 1) per non creare tagli su θ = 0.
- Il buffer `ph` aggiunge 4 byte per pixel ai buffer di calcolo e di vista.

## 1.9.5 — 2026-11-23 	

### Novità
- Legenda cromatica condivisa dalle viste, sotto il contenitore delle viste (desktop) oppure in un pannello «i» chiuso (mobile, tramite `makeInfo`). Disegna la LUT corrente con lo stesso mapping di `colorize` (densità, fase, ciclicità). Asse orizzontale in μ (iterazioni continue, 0 → maxIt della vista attiva); in modo «Equalizzato» l'asse è in percentile. Su Newton una fascia per radice.
- Tacche: `0`, periodo del ciclo colore (se completato entro l'asse, con tacche minori per i cicli successivi), `maxIt` (in percentile: `100 %`). Il raggio di fuga non è una tacca.
- Campione separato «non diverge entro N iterazioni» (Newton/Nova: «non converge…») con il colore interno (tinta unita) o con la rampa e l'indicazione del modo interno (trappola punto / croce).
- Indicatore sulla barra: la posizione e il valore di μ del punto sotto il cursore o del marker della vista attiva (stesso `evalPoint` del cursore informativo). Per i punti interni si evidenzia il campione interno; per divergenza/singolarità (Newton) nessun indicatore.
- Blocco «i» collassabile «criteri di arresto e colori»: il calcolo si ferma a maxIt per limitare il tempo, non perché il punto sia «stabile»; lettura dei colori adattata alla famiglia (Mandelbrot/Julia, altre famiglie con fuga, Newton, Nova).
- Famiglie con `caps.escape === false` (Buddhabrot, IFS): barra senza scala numerica, con la didascalia pertinente; blocco «i» e campione interno nascosti.
- Lo stato JSON/URL non cambia: la legenda deriva solo dallo stato esistente.
- Su desktop l'altezza delle viste è ridotta di 80 px per far posto alla legenda.

### Correzioni
- Nessuna.

### Limiti noti
- La barra mostra la rampa di base (luminosità 1, senza luce e rilievo); in Neon e Steli non rende luminosità e steli.
- L'asse in μ usa il maxIt della vista attiva. Con «Iterazioni automatiche» altre viste possono avere un maxIt diverso: un μ oltre la fine della barra viene fissato al bordo destro, ma il valore numerico resta quello reale.
- In modo «Equalizzato» il percentile dell'indicatore usa la CDF della vista in cui si trova il puntatore o il marker.
- I colori di divergenza (rosso) e singolarità (magenta) di Newton non hanno un campione nella legenda.
- Con le viste a schermo intero la legenda, che sta fuori dal contenitore, non è visibile.

## 1.9.0 — 2026-11-23 

### Novità
- `evalPoint(fam, params, z0x, z0y, cx, cy, maxIt)`: valutatore puntuale su tutte le famiglie a bande (Mandelbrot/Multibrot, Julia, Burning Ship e varianti, Tricorno, Newton polinomiale e trascendente, Nova, Phoenix, Magnet I e II, esponenziale, seno). Restituisce `{ kind, n, mu, zx, zy, mod, arg, de, root }`. Condivide `computeBand` (strumentazione opzionale, nessun effetto sul calcolo a bande).
- `evalPointParityTest(500)` da console: confronta `evalPoint` con `computeBand` sullo stesso pixel (kind, n, mu, de).
- Cursore informativo nell'HUD (e nel marker touch): coordinate Re/Im, stato, `n` e `mu`, `|zₙ|` (log10 se > 10⁶), `arg zₙ` in radianti e gradi. Calcolo on demand, senza nuovi buffer, con throttling a 50 ms e sospeso durante pan, zoom e selezione.
- `caps` per ogni famiglia (`escape`, `phase`, `point`, `profile`, `grid`, `newtonThresholds`, `juliaPick`) e `applyCaps()`: i controlli marcati `data-cap` si nascondono se non pertinenti. Chiamata a ogni cambio famiglia, dopo `applyState` e all'avvio.
- Con «Interno» e «Modo esterno» nascosti per IFS e Buddhabrot (inerti per quelle famiglie).

### Correzioni
- Nessuna.

### Limiti noti
- Per i punti che non divergono lo stato è «non diverge entro N iterazioni»: vicino al bordo può divergere aumentando N.
- `mu` della banda è memorizzato in Float32: il test confronta `mu` dopo arrotondamento a Float32 (tolleranza 1e-9 sul valore memorizzato).
- Nessun modo di fase né UI per le soglie Newton esistono in 1.8.5: i flag `phase` e `newtonThresholds` sono definiti ma non governano ancora controlli.
- Nella famiglia Burning Ship in modalità parametro il piano mostrato è speculare (Im c = −Im del piano); Re/Im del cursore sono le coordinate del piano.


## 1.8.5 — 2026-11-22 

### Novità
- Pulsante «i» (`aria-expanded`, `aria-controls`, area di tocco ≥ 40 px su mobile) che apre/chiude le didascalie: introduzione generale, descrizione della famiglia attiva e nota di «Luce e rilievo». Testi invariati, solo racchiusi. Chiuse all'avvio su mobile; su desktop resta aperta solo l'introduzione. `Esc` le chiude.
- Helper riutilizzabile `makeInfo(id, html, opts)` (per legenda 1.9.5 e guida 1.13.5).
- Griglia del piano di Gauss: pulsante «Griglia» e scorciatoia `G`, spenta all'avvio. Disegnata sull'overlay di ogni vista (nessun ricalcolo): assi Re/Im, origine con marcatore rosso, tacche e valori con passo 1-2-5, etichette senza sovrapposizioni, contorno doppio. Con `rot ≠ 0` gli assi ruotano e le etichette restano orizzontali. Se un asse è fuori vista, tacche ed etichette si appoggiano al bordo.
- Stato JSON/URL: campo opzionale `grid: true` (omesso se falso). Assente = spenta.
- Proprietà `axes` sulle famiglie (`'complex'`; `'none'` per IFS e Buddhabrot, dove il pulsante è nascosto e la griglia non si disegna).

### Correzioni
- Nessuna.

### Limiti noti
- Il passo 1-2-5 dà tacche ogni 80–200 px (non sempre 80–120).
- A zoom molto profondo le etichette lunghe si diradano per evitare sovrapposizioni.
- Con `rot ≠ 0` e asse fuori vista le tacche appoggiate al bordo possono mancare.
- La griglia non entra nella cronologia («Indietro» non la ripristina).
- La scorciatoia `G` non è elencata nel testo dell'introduzione (testo non riscritto).


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