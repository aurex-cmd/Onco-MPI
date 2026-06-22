# Onco-MPI

Calcolatore web dell'**indice prognostico multidimensionale oncologico (Onco-MPI)** per la stima della mortalità a 1 anno nel paziente oncologico anziano (≥70 anni), basato sulla valutazione geriatrica multidimensionale (CGA).

App installabile (PWA), mobile-first, **funzionante offline**, senza dipendenze esterne.

---

## A cosa serve

L'Onco-MPI combina 12 domini della CGA in un unico punteggio normalizzato (0–1) che colloca il paziente in una delle tre classi di rischio RECPAM, ciascuna associata a una mortalità osservata a 1 anno:

| Classe | Punteggio | Mortalità a 1 anno |
|--------|-----------|--------------------|
| Basso  | 0,00–0,46 | 2,1 %  |
| Medio  | 0,47–0,63 | 17,7 % |
| Alto   | 0,64–1,00 | 80,8 % |

Domini in input: età, sesso, BMI, ADL, IADL, ECOG PS, MMSE, n° comorbilità severe (CIRS gr. 3–4), n° farmaci, stadio del tumore, sede del tumore, presenza di caregiver.

> **Strumento di supporto decisionale.** Non sostituisce la valutazione clinica complessiva né definisce da solo l'indicazione al trattamento.

---

## Metodo e nota sull'implementazione

Il punteggio applica fedelmente la **Tabella 3** del paper originale:

```
R = Σ (Sᵢ · Dᵢ)
Onco-MPI = (R + 2,371) / 8,034   → troncato a [0, 1]
```

dove `Sᵢ` sono i pesi (coefficienti di regressione = ln(HR) del modello di Cox multivariato) e `Dᵢ` i valori dei domini. Le soglie di classe derivano dall'algoritmo RECPAM. MMSE è trattato come variabile continua (0–30), come indicato in Tabella 3.

**Avvertenza importante.** Gli esempi clinici discorsivi presenti nel testo dell'articolo **non sono riproducibili** con i coefficienti della Tabella 3. In particolare, l'esempio sull'ADL (passaggio da 6/6 a 5/6 con punteggio da 0,54 a 0,72, Δ≈0,18) è incompatibile con il peso ADL (−0,07717), che per una variazione di 1 punto produce un Δ del punteggio normalizzato di circa 0,01. Anche il primo esempio (donna di 80 anni, colorettale stadio III) calcolato con la formula della Tabella 3 restituisce ~0,20–0,25 anziché 0,44. I coefficienti della Tabella 3 sono invece internamente coerenti con i ln(HR) del modello multivariato (Tabella 4). **Questo calcolatore segue la Tabella 3; i valori numerici degli esempi narrativi vanno considerati con cautela.**

Discriminazione del modello originale: C-index 0,869 (IC 95% 0,841–0,897); calibrazione: Hosmer–Lemeshow p = 0,854.

---

## Struttura dei file

| File | Descrizione |
|------|-------------|
| `index.html` | App completa (HTML + CSS + JS in un unico file) |
| `manifest.webmanifest` | Metadati PWA (nome, icone, colori, display) |
| `sw.js` | Service worker per il funzionamento offline |
| `icon-192.png`, `icon-512.png` | Icone applicazione |
| `icon-maskable-512.png` | Icona maskable (Android) |
| `apple-touch-icon.png` | Icona iOS (180×180) |

Tutti i file devono risiedere **nella stessa cartella**.

---

## Pubblicazione (GitHub Pages)

1. Caricare tutti i file nel repository, nella stessa cartella.
2. **Settings → Pages → Build and deployment** → Source: *Deploy from a branch*, branch `main`, cartella `/ (root)`.
3. L'app sarà disponibile all'URL del repository (es. `https://aurex-cmd.github.io/onco-mpi/`).

GitHub Pages serve i contenuti via HTTPS: il service worker e l'installazione PWA funzionano senza configurazione aggiuntiva.

### Installazione su dispositivo

- **Android (Chrome):** menu ⋮ → *Aggiungi a schermata Home / Installa app*.
- **iOS (Safari):** Condividi → *Aggiungi a Home*.

Dopo la prima apertura l'app resta utilizzabile anche senza connessione.

---

## Aggiornare l'app

Il service worker usa una cache versionata. Dopo ogni modifica a `index.html` (o agli altri file):

1. Aprire `sw.js`.
2. Incrementare la versione: `const CACHE_VERSION = "onco-mpi-v2";` → `"onco-mpi-v3"`.
3. Effettuare il commit.

Al successivo accesso online, i dispositivi scaricano la nuova versione ed eliminano la cache precedente. È l'**unico passaggio manuale** da ricordare a ogni aggiornamento.

---

## Fonte

Brunello A, Fontana A, Zafferri V, Panza F, Fiduccia P, Basso U, Copetti M, Lonardi S, Roma A, Falci C, Monfardini S, Cella A, Pilotto A, Zagonel V. *Development of an oncological-multidimensional prognostic index (Onco-MPI) for mortality prediction in older cancer patients.* **J Cancer Res Clin Oncol** 2016;142(5):1069–1077. doi:10.1007/s00432-015-2088-x

Articolo pubblicato in open access (CC BY 4.0).

---

## Licenza

Codice di questo strumento: a discrezione dell'autore del repository. L'algoritmo e i coefficienti derivano dall'articolo open access sopra citato.
