# Prompt Architetturali per Slide Framework Editor

Ogni modulo è un prompt da mandare!

### Modulo 1: Architettura Core, Configurazione e Gestione Media
Agisci come Senior Web Developer. Modifica il file `presentation_editor.html` (basato su File System Access API) per centralizzare l'architettura:
1. **Stile Centralizzato:** All'apertura di una cartella root, il sistema deve cercare o creare una singola cartella `style_and_functions` contenente `slideui.css` e `slideui.js` validi per l'intero deck, eliminando la ricerca iterativa nelle sottocartelle.
2. **Configurazione JSON:** Implementa la lettura/scrittura di `presentation-config.json` nella root. Deve contenere: impostazioni numeratore (posizione e stile), ID tema, regole PDF, e una sezione `ai_prompts` contenente le istruzioni base e gli esempi HTML da passare al LLM per la generazione delle slide.
3. **Gestione Media:** Il sistema deve creare automaticamente una cartella `assets/` nella directory di ogni gruppo di slides. Configura il sistema in modo che l'utente e l'AI possano referenziare le immagini locali usando percorsi relativi (es. `/assets/logo.png`) nelle slide.

### Modulo 2: Salvataggio Esplicito e Live Preview
Riscrivi la logica di gestione dello stato e del FileSystem nell'editor:
1. **Stop Autosave:** Rimuovi la funzione `scheduleSave` che salva automaticamente su disco ogni 0.5s. Il salvataggio dei file HTML deve avvenire *solo* quando l'utente preme `Ctrl+S` o un apposito bottone "Salva".
2. **Prompt di Cambio Slide:** Se l'editor ha modifiche non salvate (dirty state) e l'utente clicca su un'altra slide o tenta di chiudere la pagina, mostra un prompt nativo (`confirm`) chiedendo se desidera salvare le modifiche prima di procedere.
3. **Preview Indipendente:** L'iframe della live preview deve continuare ad aggiornarsi in tempo reale (on change dell'editor Ace), leggendo il buffer in memoria e non il file su disco, garantendo l'anteprima istantanea anche delle modifiche non salvate.
4. **Cestino Logico:** Implementa un cestino in memoria per rinomine e cancellazioni delle slide. Scrivi fisicamente i drop/delete su disco solo alla conferma esplicita o al salvataggio globale (Undo/Redo a livello FS).

### Modulo 3: UI, UX Sidebar e Presenter View (Speaker Notes)
Aggiorna l'interfaccia utente (TailwindCSS) e la logica di presentazione:
1. **Sidebar Dinamica:** Rendi l'elenco slide a sinistra ridimensionabile (drag handler orizzontale). Colora il background degli elementi `<li>` in base alla cartella padre usando una palette tenue mappata univocamente ai nomi delle directory. Aggiungi un bottone "Nuova Cartella/Capitolo" per creare directory nativamente.
2. **Presenter View:** Sostituisci il "Present" overlay attuale. Al click, usa `window.open()` per aprire una nuova finestra (schermo del relatore) e manda la presentazione in fullscreen nello schermo secondario.
3. **Comunicazione tra Finestre:** La finestra originale controllerà quella nuova via `postMessage`. Nella vista relatore, mostra: slide attuale, slide successiva, timer, e il testo estratto da un eventuale tag `<aside class="notes">` presente nell'HTML della slide corrente.

### Modulo 4: Motore CSS e Transizioni Centralizzate
Modifica il file generatore di `slideui.js` e la logica dell'iframe:
1. **Gestore Transizioni:** Inietta nel `slideui.js` un sistema per gestire le animazioni di transizione (fade, slide-in, dissolve) modificando le classi sul `<body>` o sul wrapper principale della slide durante i comandi 'next/prev'.
2. **Parametrizzazione:** Il tipo e la durata della transizione devono essere letti dal `presentation-config.json` e passati a `slideui.js` tramite l'oggetto `window.__SLIDEUI`.
3. **Indipendenza Strutturale:** Assicurati che le classi di transizione non rompano il posizionamento assoluto (`1280x720px`) o il layout a flexbox descritto nel CSS.

### Modulo 5: Esportazione PDF Avanzata (Unrolling)
Crea un nuovo motore di esportazione `exportToPDF()`:
1. **Unrolling dei Data-Step:** Per ogni slide contenente attributi `data-step="1"`, `data-step="2"`, duplica dinamicamente l'HTML della slide 'N' volte per creare documenti piatti adatti alla stampa.
2. **Visibilità Sequenziale:** Sulla copia 1, inietta CSS per nascondere (`opacity: 0`) step >= 1. Sulla copia 2, mostra step 1 e nascondi step >= 2, e così via fino alla slide completa.
3. **Edge Cases da Configurazione:** Leggi le regole per gestire elementi che spariscono o animazioni particolari dalla chiave `pdfExportRules` del `presentation-config.json`.
4. **Stampa:** Unisci tutto in un blob HTML incollonato con la regola `@media print { page-break-after: always; }`.

### Modulo 6: AI Ecosistema e `for-ai.md`
Genera l'ecosistema di supporto per gli LLM esterni:
1. **Documentazione:** Crea il contenuto di un file `for-ai.md` da mettere nella repo. Deve documentare rigorosamente l'architettura (1280x720 fisso, uso *esclusivo* di classi utility centralizzate come `.grid-2`, `.text-accent`, `.panel`, e referenze immagini locali in `../assets/`).
2. **Iniezione Prompt:** Scrivi la logica JavaScript per estrarre la sezione `ai_prompts` dal `presentation-config.json`. Quando l'utente preme "Copy AI Prompt", l'editor deve copiare nella clipboard un prompt completo unendo le istruzioni generali del JSON, gli snippet di layout di esempio (es. math-block o immagini), e il contenuto testuale del `slideui.css` per dare il contesto di design esatto all'AI.