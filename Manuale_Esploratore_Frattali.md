---
title: Manuale — Esploratore di Frattali
copyright: "© 2026 Stefano Vesco (IK0VCK) - CatSW. All rights reserved."
author: IK0VCK
type: manual
product: fractal-explorer
license: MIT
updated: 2026-11-26
---
# Manuale — Esploratore di Frattali

- code name: **fractal-explorer**
- versione documentata: **1.17.0**

Questo manuale spiega nel dettaglio come utilizzare **Esploratore di Frattali** (`fractal-explorer`), illustrando ogni comando dell'interfaccia e chiarendo in modo semplice i concetti matematici e grafici che regolano le immagini generate.

---

## Indice
1. [Come funziona un frattale nell'app (concetti base)](#1-come-funziona-un-frattale-nellapp-concetti-base)
2. [Struttura dell'interfaccia e layout adattivi](#2-struttura-dellinterfaccia-e-layout-adattivi)
3. [Navigazione, gesti, marker e scorciatoie da tastiera](#3-navigazione-gesti-marker-e-scorciatoie-da-tastiera)
4. [Viste multiple e cornici di orientamento](#4-viste-multiple-e-cornici-di-orientamento)
5. [Il pannello Frattale: famiglie, parametri e qualità](#5-il-pannello-frattale-famiglie-parametri-e-qualità)
6. [Il pannello Colore e la Legenda cromatica](#6-il-pannello-colore-e-la-legenda-cromatica)
7. [Il pannello Luce e rilievo](#7-il-pannello-luce-e-rilievo)
8. [Analisi avanzata: cursore, griglia e profilo sezionale](#8-analisi-avanzata-cursore-griglia-e-profilo-sezionale)
9. [Zoom profondo (precisione estesa e perturbazione)](#9-zoom-profondo-precisione-estesa-e-perturbazione)
10. [Motore di calcolo, scheda grafica (WebGL2) e condivisione](#10-motore-di-calcolo-scheda-grafica-webgl2-e-condivisione)
11. [Guida didattica integrata](#11-guida-didattica-integrata)

---

## 1. Come funziona un frattale nell'app (concetti base)

Per sfruttare al meglio tutti i controlli è utile avere chiari quattro concetti ricorrenti nell'app:

- **Piano complesso ($x + iy$):** l'area di disegno rappresenta una porzione del piano dei numeri complessi. L'asse orizzontale è la **parte reale** ($\text{Re}$), mentre l'asse verticale è la **parte immaginaria** ($\text{Im}$). Ogni pixel corrisponde quindi a un numero complesso.
- **Iterazione e orbita:** per quasi tutte le famiglie, il programma prende il numero complesso associato al pixel e gli applica ripetutamente una formula matematica (un'iterazione dopo l'altra). La sequenza dei valori ottenuti passo dopo passo si chiama **orbita**.
- **Fuga (esterno) e prigionia (interno):**
  - Se durante le iterazioni la distanza del punto dall'origine supera una soglia di sicurezza (**raggio di fuga**), il punto "scappa" verso l'infinito: fa parte dell'**esterno** del frattale. Il numero di passi impiegati per scappare determina il colore.
  - Se il punto continua a rimbalzare senza mai superare la soglia entro il numero massimo di passi consentito (**Iterazioni base**), viene considerato intrappolato: fa parte dell'**interno** del frattale (di solito colorato di nero o con trappole geometriche).
- **Iterazioni continue ($\mu$):** contare solo i passi interi ($1, 2, 3\dots$) creerebbe scalini netti tra un colore e l'altro. L'app calcola invece un valore decimale continuo chiamato **$\mu$** (*mu*), che stima la frazione di passo con cui l'orbita ha superato il raggio di fuga, rendendo le sfumature perfettamente lisce.

> **Nota sulle famiglie a densità e geometriche:** alcune famiglie (come *Buddhabrot*, *IFS*, *Attrattori strani*, *Biforcazione* e *Geometrici*) non colorano ogni pixel in base a una fuga, ma accumulano milioni di punti o segmenti su una griglia di densità per mostrare la forma complessiva del sistema.

---

## 2. Struttura dell'interfaccia e layout adattivi

L'app organizza i suoi elementi principali in quattro zone:
1. **Intestazione superiore:** mostra il titolo, il pulsante **Condividi** (per copiare un link con lo stato attuale) e il collegamento **Guida - Ver. 1.17.0** che apre la Guida didattica in fondo alla pagina.
2. **Area delle viste:** lo spazio centrale dove vengono disegnati il frattale, l'HUD (le informazioni sovraimpresse in alto a sinistra di ogni vista), l'eventuale griglia, il marker e il segmento del profilo.
3. **Legenda cromatica:** collocata sotto le viste, traduce i colori sullo schermo in valori numerici di $\mu$, percentili o radici.
4. **Barra strumenti e Pannello controlli:** racchiudono tutti i comandi operativi, suddivisi nelle schede **Frattale**, **Colore** e **Luce e rilievo**.

L'interfaccia rileva automaticamente le dimensioni dello schermo e adotta tre disposizioni (**layout adattivi**):
- **Schermo ampio (`wide`, larghezza $\ge 900\text{ px}$ in orizzontale):** il pannello dei controlli è sempre visibile sulla destra, affiancato all'area delle viste e alla barra strumenti. Puoi comunque nasconderlo temporaneamente premendo `P` per avere più spazio.
- **Schermo verticale stretto (`compact`, larghezza $< 900\text{ px}$, tipico degli smartphone):** le viste occupano quasi tutto lo schermo e in basso compare una barra fissa con i 5 comandi essenziali (**Vista iniziale**, **Indietro**, **Zoom area**, **Viste** `1`/`2`/`4` e **Controlli**). Toccando **Controlli**, si apre dal basso un pannello a schede (**Frattale**, **Colore**, **Luce** e **Altro**) che puoi ridimensionare, chiudere con `✕` o `Esc`, oppure abbassare trascinando la maniglia superiore.
- **Schermo orizzontale basso (`short`, altezza $< 500\text{ px}$):** la barra strumenti si posiziona verticalmente sul lato sinistro e il pannello dei controlli scorre da destra quando premi **Controlli**.

Nella scheda **Altro** (presente nelle modalità `compact` e `short`) trovano posto tutti gli strumenti secondari che su schermo ampio stanno nella barra: **Cornici**, **Griglia**, i comandi del **Profilo**, **Julia dal marker**, **Anima colori**, **Salva PNG**, **Esporta JSON** e il numero di versione.

> **Icone informative (`i`):** accanto alle sezioni principali trovi piccoli pulsanti `i` che aprono brevi spiegazioni contestuali. Su schermo ampio (`wide`) le spiegazioni introduttive, della famiglia attiva e della luce partono già aperte; su schermi piccoli (`compact` e `short`) partono chiuse per non occupare spazio.

---

## 3. Navigazione, gesti, marker e scorciatoie da tastiera

### Spostarsi e ingrandire con il mouse
- **Zoom sul puntatore:** gira la **rotella del mouse**. L'ingrandimento avviene mantenendo fisso il punto sotto la freccia.
- **Zoom rapido $\times 2$:** fai **doppio clic** con il tasto sinistro in un punto qualsiasi della vista.
- **Zoom su rettangolo:** tieni premuto il **tasto sinistro** e trascina per disegnare un riquadro di selezione, poi rilascia per ingrandire esattamente quell'area. Premi `Esc` prima di rilasciare se vuoi annullare la selezione (se il rettangolo è troppo piccolo, lo zoom non viene applicato per evitare clic accidentali).
- **Spostare la vista (Pan):** trascina tenendo premuto il **tasto destro**, il **tasto centrale (rotella)** oppure tenendo premuto il tasto **Spazio** mentre trascini con il sinistro. Anche una pressione prolungata con il tasto sinistro prima di muovere il mouse attiva lo spostamento.

### Spostarsi e ingrandire su schermo touch
- **Spostare la vista (Pan):** appoggia **un dito** e trascina oltre circa 10 pixel. Durante il trascinamento l'immagine si sposta istantaneamente e viene ricalcolata non appena sollevi il dito.
- **Pizzico (Pinch-to-zoom):** appoggia **due dita** e allargale o avvicinale per ingrandire o ridurre, spostando contemporaneamente l'inquadratura.
- **Zoom rapido $\times 2$:** fai **doppio tocco** rapido su un punto.
- **Zoom su rettangolo:** tocca il pulsante **Zoom area** nella barra strumenti, poi trascina un dito sulla vista per tracciare il riquadro desiderato.
- **Menu contestuale rapido:** tieni premuto **un dito** fermo per circa mezzo secondo (~450 ms) per aprire un menu contestuale rapido sulla vista.

### Il Marker e la funzione «Julia dal marker»
Il **marker** è un mirino ancorato a una coordinata precisa del piano complesso:
- **Piazzare o spostare il marker:** fai un **clic singolo** (o un **tocco breve** inferiore a 250 ms) senza trascinare. Il marker si posiziona in quel punto e nell'HUD compaiono i suoi valori esatti.
- **Rimuovere il marker:** fai clic o tocca nuovamente nelle immediate vicinanze del marker (entro 24 pixel).
- **Usare «Julia dal marker»:** quando stai esplorando l'insieme di **Mandelbrot**, ogni punto $c$ del piano corrisponde a un diverso insieme di **Julia**.
  - Se hai già piazzato un marker su una vista Mandelbrot, premendo **Julia dal marker** l'app passa alla famiglia *Julia* impostando come parametro $c$ esattamente le coordinate del marker.
  - Se non hai ancora piazzato un marker, premendo **Julia dal marker** il pulsante entra in attesa (*«Scegli il punto sulla vista…»*): il tuo successivo clic o tocco sulla vista sceglierà il punto e aprirà subito il corrispondente frattale di Julia (premere di nuovo il pulsante annulla l'attesa).

### Cronologia e scorciatoie da tastiera
Ogni volta che cambi inquadratura, parametro o famiglia, l'app salva il passaggio nella cronologia della sessione. Puoi muoverti rapidamente con questi tasti:

| Tasto | Azione |
| :--- | :--- |
| `+` / `−` | Aumenta o riduce lo zoom al centro della vista attiva |
| `R` | **Vista iniziale**: riporta la vista attiva all'inquadratura predefinita della famiglia |
| `B` | **Indietro**: annulla l'ultimo spostamento o ripristina lo stato precedente |
| `F` | **Schermo intero**: attiva o disattiva la modalità a tutto schermo del browser |
| `P` | Mostra o nasconde il pannello dei controlli (su schermo ampio `wide`) |
| `G` | Attiva o disattiva la **Griglia** del piano complesso |
| `O` | Attiva o disattiva lo strumento **Profilo** sezionale |
| `Esc` | Chiude selezioni rettangolari in corso, pannelli aperti o didascalie |

---

## 4. Viste multiple e cornici di orientamento

Puoi dividere l'area centrale in **1, 2 o 4 viste** indipendenti (`V1`, `V2`, `V3`, `V4`) tramite il selettore **Viste** nella barra strumenti:
- **Come funzionano:** tutte le viste mostrano la stessa famiglia di frattali con gli stessi colori, ma ognuna conserva **inquadratura, zoom, marker e profilo indipendenti**.
- **Vista attiva:** è evidenziata da un bordo colorato e da un badge identificativo (`V1`–`V4`) dello stesso colore. I comandi da tastiera, **Salva PNG** e il pulsante **Julia dal marker** agiscono sempre sulla vista attiva (basta un clic/tocco su un'altra vista per renderla attiva).
- **Aggiungere o chiudere viste:** passando da 1 a 2 (o da 2 a 4), le nuove viste vengono create duplicando l'inquadratura della vista attiva, così puoi iniziare subito a zoomare su un dettaglio mantenendo l'altra vista come riferimento. Per chiudere una singola vista, premi il pulsante `✕` nel suo angolo (disabilitato quando resta una sola vista).
- **Adattamento automatico su schermi piccoli:** su schermi stretti ($< 700\text{ px}$) le viste vengono impilate verticalmente fino a un massimo di 3; su schermi orizzontali bassi ($< 500\text{ px}$) vengono affiancate al massimo 2 viste. Se riduci la finestra, le viste in eccesso vengono chiuse con un avviso e possono essere recuperate con `B` (**Indietro**).
- **Cornici proiettate:** attivando l'interruttore **Cornici**, ogni vista disegna al proprio interno i rettangoli che rappresentano l'inquadratura delle altre viste aperte, completi di etichetta (`V1`–`V4`). Se l'inquadratura di un'altra vista cade fuori dallo schermo, appare una freccia direzionale lungo il bordo per indicarti in che direzione si trova.

---

## 5. Il pannello Frattale: famiglie, parametri e qualità

Il pannello **Frattale** controlla la formula matematica da calcolare e la precisione del disegno. I controlli visibili si adattano automaticamente alla famiglia scelta (ad esempio, le opzioni sulle iterazioni non compaiono per i frattali geometrici).

### Controlli generali
- **Famiglia:** permette di scegliere tra le 15 famiglie disponibili.
- **Preset:** per ogni famiglia offre una raccolta di configurazioni curiose e punti famosi già pronti da esplorare (es. *Valle dei cavallucci marini*, *Mini-Mandelbrot*, *Coniglio di Douady*, *Felce di Barnsley*).
- **Parametri specifici:** cursori e campi numerici che cambiano in base alla famiglia (potenza $d$, parte reale e immaginaria di $c$, variante geometrica, angolo o fattori di scala).
- **Iterazioni base:** stabilisce il numero massimo di passi prima di dichiarare un punto "interno". Valori bassi rendono il calcolo velocissimo ma arrotondano i dettagli fini; valori alti rivelano filamenti sottili e spirali profonde.
- **Iterazioni automatiche con lo zoom (consigliato):** quando è attivo, l'app aumenta progressivamente il tetto di iterazioni man mano che scendi in profondità con lo zoom, evitando che il frattale diventi un blocco nero piatto senza dover regolare ogni volta il cursore a mano.
- **Qualità:** regola la risoluzione interna di calcolo rispetto ai pixel dello schermo:
  - *Veloce*: calcola meno pixel per offrire la massima fluidità durante l'esplorazione rapida.
  - *Normale*: rapporto 1:1 tra pixel calcolati e schermo (equilibrio ideale).
  - *Alta* e *Ultra*: calcolano più campioni per pixel (supersampling) per eliminare seghettature e rumore sui dettagli finissimi, richiedendo più tempo di elaborazione.

### Le 15 famiglie disponibili

#### A. Famiglie a bande di fuga (piano complesso)
1. **Mandelbrot e Multibrot:** applica $z_{n+1} = z_n^d + c$ partendo da $z_0 = 0$, dove $c$ è la coordinata del pixel. Con $d = 2$ si ottiene il celebre insieme di Mandelbrot; aumentando $d$ ($3, 4\dots$) si ottengono i *Multibrot* con simmetria rotazionale di ordine $d - 1$.
2. **Julia:** usa la stessa formula $z_{n+1} = z_n^d + c$, ma qui $c$ è un valore fisso scelto dai parametri (o dal marker) e il pixel determina il punto di partenza $z_0$.
3. **Burning Ship e varianti:** prima di elevare al quadrato prende il valore assoluto delle componenti reale e/o immaginaria, creando strutture spigolose simili a navi o architetture. Include le varianti *Burning Ship*, *Celtic*, *Perpendicolare*, *Nave perpendicolare* e *Heart*, sia in modalità mappa (come Mandelbrot) sia in modalità *Julia*.
4. **Tricorno e Multicorn:** utilizza il numero complesso coniugato ($z_{n+1} = \bar{z}_n^d + c$), producendo figure a tre punte (per $d = 2$) e simmetrie speculari caratteristiche.
5. **Newton:** visualizza i bacini di attrazione del metodo di Newton ($z_{n+1} = z_n - a \cdot f(z_n)/f'(z_n)$) per trovare gli zeri di una funzione. Funziona sia su polinomi ($z^3 - 1$, $z^4 - 1$, ecc.) sia su funzioni trascendenti ($\sin z$, $\cos z$, $\tan z$, $\sinh z$, $\cosh z$, $e^z - a$, $\sin z - a \cdot z$). I colori distinguono quale radice viene raggiunta e in quanti passi; il parametro complesso $a$ permette di introdurre il "rilassamento" generando frattali di Nova generalizzati.
6. **Nova:** combina il metodo di Newton con l'aggiunta del termine $+c$ ad ogni passo, partendo da $z_0 = 1$, fondendo i bacini di radici con le strutture del Mandelbrot.
7. **Phoenix:** introduce una "memoria" del passo precedente ($z_{n+1} = z_n^2 + p + q \cdot z_{n-1}$), generando piume e simmetrie intrecciate regolabili tramite $p$ e $q$.
8. **Magnet I e Magnet II:** formule razionali (con numeratore e denominatore) nate nello studio delle transizioni di fase magnetiche. Uniscono zone di fuga all'infinito con bacini interni di attrazione verso $z = 1$, disponibili sia in modalità parametro sia in modalità Julia.
9. **Esponenziale:** calcola $z_{n+1} = c \cdot e^{z_n}$ (con fuga rilevata quando $\operatorname{Re}(z) > 50$), generando rami arborescenti e bande orizzontali ripetute.
10. **Seno:** calcola $z_{n+1} = c \cdot \sin(z_n)$ (con fuga rilevata quando $|\operatorname{Im}(z)| > 50$), producendo catene simmetriche di bulbi e filamenti.

#### B. Famiglie a densità, sistemi dinamici e geometrici
11. **Buddhabrot (traiettorie):** invece di colorare il punto di partenza, traccia l'intera traiettoria dei punti che sfuggono dal Mandelbrot e illumina tutti i pixel attraversati lungo il percorso, rivelando una figura simile a una silhouette meditante.
12. **IFS e chaos game:** costruisce frattali auto-simili applicando casualmente milioni di trasformazioni geometriche (rotazioni, riduzioni, traslazioni). Include *Felce di Barnsley*, *Albero*, *Drago di Heighway*, *Curva C di Lévy* e *Triangolo di Sierpiński*.
13. **Attrattori strani:** traccia l'orbita di sistemi dinamici caotici (*Clifford*, *de Jong*, *Hénon*), mostrando le trame setose su cui si adagia il sistema nel tempo. Quando disponibile, nell'HUD viene mostrato anche l'**esponente di Lyapunov massimo** (un numero positivo conferma il comportamento caotico).
14. **Biforcazione (mappa logistica):** mostra come il comportamento a lungo termine della mappa $x_{n+1} = r \cdot x_n (1 - x_n)$ passi da un singolo valore stabile al raddoppio del periodo fino al caos al variare del parametro $r$ (o sull'asse coniugato $c$). Include linee guida opzionali e griglia `rx`.
15. **Geometrici (Koch, albero pitagorico):** disegna per suddivisione ricorsiva la *Curva di Koch*, il *Fiocco di Koch* e l'*Albero pitagorico*, adattando automaticamente il livello di dettaglio allo zoom (tagliando i segmenti più piccoli di circa mezzo pixel per mantenere prestazioni istantanee).

---

## 6. Il pannello Colore e la Legenda cromatica

Il pannello **Colore** trasforma i dati numerici calcolati dal motore in immagini leggibili ed espressive.

### Stile pronto e Palette
- **Stile pronto:** applica con un clic combinazioni studiate di palette, modo esterno, fonte colore e luce/rilievo.
- **Palette:** sceglie la scala cromatica. Tutte le sfumature sono interpolate nello spazio colore **OKLCH**, che mantiene costante la luminosità percepita dall'occhio umano evitando macchie sbiadite o passaggi bruschi. Scegliendo **Personalizzata**, compaiono i selettori colore per creare la tua gradazione.
- **Ripetizione / Scala e Fase:** regolano quante volte la palette si ripete lungo le bande di fuga e ne fanno scorrere i colori. Con **Anima colori** puoi avviare uno scorrimento continuo e fluido della fase.

### Modo esterno (come vengono distribuite le sfumature)
- **Sfumato continuo:** assegna il colore in modo proporzionale alle iterazioni continue $\mu$. È il metodo più naturale e fedele al gradiente di fuga.
- **Equalizzato:** ridistribuisce i colori in base alla frequenza statistica dei valori presenti nella vista attuale (tramite funzione di distribuzione cumulativa, CDF). È utilissimo negli zoom molto profondi, dove quasi tutti i pixel hanno valori di iterazione altissimi e vicini tra loro, perché restituisce subito pieno contrasto a tutta l'immagine.
- **Neon (filamenti):** utilizza la **stima di distanza** (Distance Estimator, analitica o numerica) per illuminare solo i bordi sottilissimi del frattale su sfondo scuro, mettendo in risalto l'ossatura filamentosa invisibile con le bande classiche.
- **Steli (germogli):** evidenzia le linee lungo cui le orbite passano vicine agli assi coordinati, facendo emergere ramificazioni simili a steli vegetali o nervature all'interno delle bande.

### Fonte colore (quale grandezza fisica colorare)
Disponibile per le famiglie che supportano l'analisi di fase:
- **Iterazioni:** usa esclusivamente la velocità di fuga $\mu$ (comportamento classico).
- **Fase finale:** colora in base all'angolo $\arg z_n$ con cui l'orbita supera il raggio di fuga. Può operare in modo continuo oppure a **2 livelli** (dividendo i punti a seconda del segno della parte immaginaria $\operatorname{Im} z_n$, creando il classico effetto a "scacchiera" lungo i rami).
- **Stripe average:** calcola una media oscillante lungo tutta l'orbita controllata dalla **frequenza `s`**, disegnando striature interne parallele ai filamenti che rivelano la geometria nascosta delle spirali.
- **Fase media (sperimentale):** combina l'angolo medio attraversato durante l'intera orbita.
- **Peso miscela (`α`):** quando scegli *Fase finale*, *Stripe average* o *Fase media*, questo cursore dosa quanto peso dare alla fase rispetto alle iterazioni classiche (miscelate in spazio **OKLab**).

### Colorazione dell'interno
- **Interno:** decide come mostrare i punti intrappolati che non sfuggono:
  - *Tinta unita*: riempie l'interno con un colore uniforme scelto tramite **Tinta interna** (di default nero).
  - *Trappola punto* / *Trappola croce*: misura quanto l'orbita interna si avvicina all'origine $(0,0)$ o agli assi coordinati, rivelando strutture simmetriche nascoste anche dentro il frattale.

### La Legenda cromatica
La barra orizzontale sotto l'area delle viste mostra in tempo reale la mappa tra colori e valori:
- Indica la scala di $\mu$ (oppure i percentili `0%–100%` in modalità *Equalizzato*, o le fasce delle singole radici nella famiglia *Newton*).
- Mostra il campione della **tinta interna** e un indicatore mobile che segna esattamente dove cade il punto sotto il cursore (o sotto il marker).
- Il pulsante `i` accanto alla legenda spiega i criteri di arresto attivi per la famiglia selezionata.

---

## 7. Il pannello Luce e rilievo

Il pannello **Luce e rilievo** tratta il valore di iterazione continua $\mu$ come se fosse l'altitudine di un terreno montuoso, calcolando in tempo reale la pendenza della superficie e l'illuminazione (modello *Blinn–Phong*):

- **Luce (intensità):** regola la forza della sorgente luminosa direzionale. A `0` l'immagine è piatta; alzando il valore compaiono luci e ombre sulle creste delle bande.
- **Azimut:** ruota di $360^\circ$ la direzione orizzontale da cui proviene la luce (come spostare il Sole attorno all'orizzonte).
- **Elevazione:** regola l'altezza della sorgente luminosa sull'orizzonte. Valori bassi radenti accentuano ogni minima increspatura; valori alti illuminano dall'alto.
- **Rilievo:** controlla quanto sono ripidi i "pendii" calcolati dalle iterazioni, aumentando o attenuando la profondità percepita dell'incisione.
- **Lucentezza:** regola i riflessi speculari, dando alla superficie un aspetto opaco come pietra oppure lucido come metallo o ceramica smaltata.
- **Cangiante:** aggiunge una variazione cromatica legata all'inclinazione della superficie rispetto all'osservatore (effetto iridescenza). Funziona anche quando l'intensità della **Luce** è impostata a `0`.

---

## 8. Analisi avanzata: cursore, griglia e profilo sezionale

Oltre all'esplorazione visiva, l'app offre tre strumenti di misura matematica:

### 1. HUD e lettura dati al cursore / marker
In alto a sinistra su ogni vista, l'HUD mostra le coordinate del centro, l'ampiezza della vista, il fattore di zoom, le iterazioni attive e, quando passi il mouse o piazzi un marker, i dati puntuali dell'orbita:
- **Coordinate ($\text{Re}, \text{Im}$):** posizione esatta sul piano complesso.
- **Stato:** indica se il punto è *Esterno* (fuggito), *Interno* (intrappolato) o, per Newton, verso quale radice o ciclo è converso.
- **Iterazioni `n` e `μ`:** numero di passi effettuati e valore continuo di fuga.
- **Modulo `|zₙ|` (o `log₁₀|zₙ|`) e Fase `arg zₙ`:** distanza dall'origine e angolo in radianti al termine del calcolo.

### 2. Griglia del piano complesso (`G`)
Premendo **Griglia** (o il tasto `G`), l'app sovrappone gli assi reale e immaginario e una griglia graduata con etichette numeriche adattate automaticamente al livello di zoom. Quando si entra nella modalità di zoom profondo (precisione estesa), la griglia viene disattivata automaticamente perché le etichette richiederebbero decine di cifre decimali.

### 3. Profilo sezionale e pannello Grafici (`O`)
Lo strumento **Profilo** permette di "tagliare" il frattale lungo una linea retta e analizzare come cambiano i valori matematici lungo quel segmento (disponibile per tutte le famiglie che supportano il campionamento puntuale):
1. **Tracciare il segmento:** attiva **Profilo** nella barra strumenti (oppure premi `O`), poi trascina sulla vista attiva dal punto iniziale **A** al punto finale **B**.
2. **Modificare il segmento:** puoi afferrare e trascinare le maniglie circolari ai capi **A** e **B** per riposizionare il taglio con precisione. *(Nota: finché il pulsante Profilo è attivo, il trascinamento con un dito o tasto sinistro sposta il segmento; per spostare la vista usa due dita, il tasto centrale del mouse oppure `Spazio` + trascinamento).*
3. **Comandi dedicati:**
   - **Centra sul segmento:** adatta automaticamente lo zoom e il centro della vista per inquadrare comodamente `A → B`.
   - **Cancella profilo:** rimuove il segmento dalla vista e chiude i grafici.
   - **Grafici:** apre o chiude il pannello con i diagrammi della sezione.
4. **Leggere i grafici:** il pannello mostra due grafici allineati sullo stesso asse orizzontale $t \in [0, 1]$ (da `A` a `B`):
   - **Grafico superiore (75% dell'altezza):** mostra l'andamento di una grandezza selezionabile tra **Iterazioni continue $\mu$**, **Potenziale $\log|z_n|/d^n$**, **$\log_{10}|z_N|$** o **Stima di distanza**, colorato con la stessa palette della vista.
   - **Grafico inferiore (25% dell'altezza):** mostra l'andamento della fase $\arg z_n \in (-\pi, \pi]$, interrompendo la linea nei punti in cui l'angolo compie un salto netto maggiore di $\pi$.
   - **Linee tratteggiate gialle:** evidenziano i punti esatti in cui il segmento attraversa il confine tra l'esterno e l'interno del frattale.
   - **Interazione e CSV:** passando il cursore o il dito sui grafici vedrai scorrere un indicatore sincronizzato direttamente lungo il segmento `A → B` sulla vista, con i valori numerici aggiornati in tempo reale. Il pulsante **Copia CSV** copia negli appunti tutti i dati campionati per poterli incollare in un foglio di calcolo.

---

## 9. Zoom profondo (precisione estesa e perturbazione)

I normali processori dei computer eseguono i calcoli in virgola mobile standard a 64 bit (**FP64**, circa 15–17 cifre decimali significative). Quando ingrandisci un frattale oltre circa **$10^{13}$ volte**, due pixel adiacenti diventano così vicini che la differenza tra le loro coordinate scende sotto la precisione di FP64: senza accorgimenti particolari l'immagine diventerebbe a grossi blocchi pixelati.

Per superare questo limite su **Mandelbrot ($d = 2$)** e **Julia ($d = 2$)**:
1. **Attivazione automatica:** non appena lo zoom supera la soglia della doppia precisione, l'app attiva automaticamente il motore di **perturbazione** e nell'HUD compare l'indicazione **Precisione: estesa** insieme al numero di cifre decimali utilizzate.
2. **Come funziona in breve:** invece di calcolare ogni singolo pixel con numeri giganteschi (che sarebbe lentissimo), l'app calcola **una sola orbita di riferimento** (al centro della vista, più l'orbita di $0$ su Julia per gestire eventuali *rebasing* quando un'orbita passa troppo vicino al punto critico) usando aritmetica intera esatta (`BigInt`) con precisione decimale adattata allo zoom. Tutti gli altri pixel vengono poi calcolati velocemente in FP64 come piccole differenze (*perturbazioni*) rispetto a quell'orbita guida.
3. **Limite massimo:** la precisione estesa accompagna lo zoom fino a un'ampiezza di **$10^{-250}$** (250 zeri dopo la virgola), punto in cui lo zoom si arresta mostrando un avviso dedicato.
4. **Altre famiglie o potenze:** sulle famiglie in cui la perturbazione non è attiva, superata la soglia di FP64 l'HUD segnala il raggiungimento del limite di precisione standard.

---

## 10. Motore di calcolo, scheda grafica (WebGL2) e condivisione

### Calcolo parallelo e anteprima progressiva
- **Web Worker:** i calcoli matematici delle orbite vengono suddivisi in fasce o riquadri (*tile*, sfruttando dove possibile l'ottimizzazione **Mariani–Silver** che riempie istantaneamente le aree interne uniformi) ed eseguiti in parallelo da più thread in background (*Web Worker*), così l'interfaccia rimane sempre reattiva.
- **Anteprima immediata:** mentre un'inquadratura complessa è in fase di calcolo ad alta risoluzione, l'app mostra subito un'anteprima a bassa risoluzione e la sostituisce progressivamente con l'immagine nitida man mano che i riquadri vengono completati.

### Accelerazione grafica WebGL2
- **Colorazione istantanea:** per tutte le famiglie a bande, i risultati numerici calcolati dai worker vengono passati alla scheda grafica tramite **WebGL2** (`GpuColor`). Questo permette di cambiare palette, ripetizione, fase, modo esterno, luci e rilievi **istantaneamente e senza dover ricalcolare le iterazioni**.
- **Ripiego automatico:** se il browser o il dispositivo non supportano WebGL2 (oppure se avvii l'app aggiungendo `?gpu=0` all'indirizzo web per diagnostica), il programma passa automaticamente alla colorazione su CPU mantenendo tutte le funzioni operative.

### Condividere, salvare ed esportare
- **Condividi:** copia negli appunti un indirizzo web contenente il parametro `?s=...`, che racchiude in forma compatta l'intero stato dell'esplorazione (famiglia, parametri, zoom, coordinate ad alta precisione `chp`, colori, luci e viste). Chiunque apra quel link vedrà esattamente la tua stessa immagine.
- **Salva PNG:** scarica sul dispositivo l'immagine della **vista attiva** alla risoluzione corrente, comprensiva di colori, luci e rilievi.
- **Esporta JSON:** apre una finestra di dialogo in cui puoi leggere e copiare l'intero stato dell'app in formato testuale **JSON**, oppure incollare un JSON salvato in precedenza e applicarlo con un clic.

---

## 11. Guida didattica integrata

Scorrendo in fondo alla pagina (o premendo **Guida - Ver. 1.17.0** nell'intestazione) accedi alla **Guida didattica**, organizzata in quattro schede a fisarmonica:
- **A · Manuale breve:** un riepilogo concettuale su navigazione, relazione tra Mandelbrot e Julia (dicotomia di Fatou–Julia), iterazioni, significato del nero, colori, cursore, griglia e profilo.
- **B · Storia e riferimenti:** dodici schede storiche che raccontano i protagonisti e le scoperte dietro i frattali e i sistemi dinamici, complete di riferimenti bibliografici e link esterni.
- **C · Applicazioni:** panoramica sull'uso pratico dei frattali fuori dalla matematica pura (analisi numerica, fisica statistica, progettazione di antenne frattali, grafica procedurale e modelli naturali).
- **D · Laboratorio:** cinque esperimenti interattivi guidati. Premendo il pulsante **Prova tu** su una scheda, l'app carica automaticamente la configurazione dell'esperimento (inclusi zoom profondo, profilo o viste multiple), scorre fino all'area di disegno e ti permette di esplorarlo subito (puoi sempre tornare indietro con `B`).
