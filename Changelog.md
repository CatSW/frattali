---
title: Changelog fractal-explorer
copyright: "© 2026 Stefano Vesco (IK0VCK) - CatSW. All rights reserved."
author: IK0VCK
type: changelog
product: fractal-explorer
license: MIT
updated: 2026-11-20
---
# Changelog — Esploratore di Frattali

Formato ispirato a [Keep a Changelog](https://keepachangelog.com/it/1.1.0/), versionamento semantico adattato:
- `0.x.0` = versioni di pre-pubblicazione (una per ogni blocco di lavoro concluso);
- `1.0.0` = prima versione pubblicata su GitHub Pages, quando sarà ritenuta sufficientemente matura.

Le date sono nel formato `AAAA-MM-GG`.

---

## [Non rilasciato]

_Vuoto: le voci vengono aggiunte al termine di ogni sessione di lavoro e spostate sotto un numero di versione alla chiusura del blocco._

---
## [0.3.0] - 2026-11-20 — Pre-pubblicazione (baseline)

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