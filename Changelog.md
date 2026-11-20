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