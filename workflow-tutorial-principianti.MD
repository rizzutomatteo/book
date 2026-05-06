# Guida Completa per Principianti — Da Zero a Sviluppatore Professionista Solo

Questa guida ti accompagna dall'inizio assoluto fino alla padronanza di un workflow professionale completo per lavorare da solo su progetti software seri. Non assume alcuna conoscenza pregressa di Git, GitHub, Docker, CI/CD, o sviluppo professionale. Si limita a presupporre che tu sappia programmare in almeno un linguaggio (anche solo le basi) e che tu sappia usare un computer.

Il documento si legge in ordine. Ogni capitolo costruisce sui precedenti. Saltare avanti porta confusione.

Tempo stimato di lettura attiva: 8-12 ore distribuite su una o due settimane. Tempo per applicare tutto su un primo progetto reale: 20-40 ore. Considera questa guida un investimento: la fatica iniziale ti restituirà mesi del tuo tempo nei progetti futuri.

---

## Indice

- **Introduzione** — chi, cosa, come
- **Parte 1** — Fondamenti concettuali (cosa devi sapere prima di iniziare)
- **Parte 2** — Setup della tua macchina
- **Parte 3** — Il tuo primo progetto: setup completo
- **Parte 4** — Il tuo primo progetto: sviluppo della prima feature
- **Parte 5** — Deploy e operazioni
- **Parte 6** — Manutenzione continua
- **Parte 7** — Glossario, cheat-sheet, risorse esterne

---

# Introduzione

## A chi serve questa guida

A chi sa programmare ma non sa ancora come si organizza un progetto software professionale. A chi ha sempre lavorato a "script che funzionano sulla mia macchina" e vuole capire cosa serve per avere un progetto deployato in produzione, mantenibile, sicuro e tracciabile.

Non è una guida per imparare a programmare. Non insegno linguaggi, algoritmi, strutture dati. Quello deve essere già nelle tue corde, anche solo a livello base.

## Cosa imparerai

Al termine sarai in grado di:

- Configurare da zero un progetto software con tutti gli strumenti professionali (Git, GitHub, Docker, CI/CD, secret management, observability).
- Sviluppare nuove feature seguendo un ciclo strutturato: scoping, ricerca, pianificazione, TDD, code review, security review, verification, PR, merge, deploy.
- Gestire il progetto in produzione: monitoraggio, backup, rollback, gestione incidenti.
- Mantenere il progetto nel tempo: aggiornamenti dipendenze, security patch, performance, costi.
- Usare Claude Code e il plugin [everything-claude-code](https://github.com/affaan-m/everything-claude-code) come collaboratore di sviluppo, sostituendo le figure (peer reviewer, security officer, architetto) che lavorando da solo non hai.

Il risultato finale è il workflow professionale documentato in `workflow-solo-professionale.md`. Questa guida ti porta a quel livello partendo da zero.

## Come usare questa guida

Leggila in ordine. Ogni concetto si appoggia ai precedenti.

Quando incontri un concetto nuovo (Git, Docker, CI/CD, ecc.), la guida lo spiega. Se senti il bisogno di approfondire oltre quanto qui scritto, **Parte 7.3** elenca risorse esterne curate.

Quando arrivi alle parti pratiche (Parte 3, 4, 5), apri il terminale e segui i comandi davvero. Leggere senza fare non funziona.

Tieni un block-notes (digitale o cartaceo) accanto. Annota ogni cosa che non capisci subito: tornaci alla fine del capitolo.

---

# Parte 1 — Fondamenti Concettuali

Prima di toccare qualsiasi tool, ti servono alcuni concetti. Ogni sezione di Parte 1 segue lo schema: **cos'è, perché ti serve, come si usa a livello concettuale, dove approfondire**.

## 1.1 — La riga di comando (terminale, shell)

### Cos'è

Il **terminale** è una finestra dove digiti comandi testuali al computer invece di cliccare. La **shell** è il programma dentro il terminale che interpreta i tuoi comandi (Bash, Zsh, PowerShell sono shell diverse).

### Perché ti serve

Tutti gli strumenti di sviluppo professionale (Git, Docker, package manager, runner di test, ecc.) hanno un'interfaccia a riga di comando. La grafica è opzionale, la riga di comando è obbligatoria. Non puoi essere uno sviluppatore senza saperla usare.

### Concetti chiave

**Working directory**: la cartella in cui ti trovi adesso. Ogni comando si applica relativamente a questa cartella se non specifichi altrimenti. Vedi sempre dove sei con `pwd` (print working directory).

**Path**: il percorso di un file o cartella. Esistono due tipi:

- **Assoluto**: parte dalla radice del filesystem. Su Linux/Mac inizia con `/` (es. `/home/utente/progetti/app`). Su Windows con `C:\` (es. `C:\Users\utente\progetti\app`).
- **Relativo**: parte dalla working directory. `./file.txt` è "file.txt nella cartella corrente". `../file.txt` è "file.txt nella cartella genitore". `progetti/app` è "la cartella app dentro la cartella progetti, a partire da dove sono".

**Variabili d'ambiente**: valori che il sistema operativo (o tu) imposta e che i programmi possono leggere. Esempio: `PATH` contiene le cartelle dove la shell cerca i programmi quando tu scrivi un comando. `HOME` è la tua cartella personale.

### Comandi base che devi imparare

```bash
pwd                        # mostra la cartella corrente
ls                         # lista file e cartelle nella cartella corrente
ls -la                     # lista anche i file nascosti (.qualcosa) con dettagli
cd nomecartella            # entra in nomecartella
cd ..                      # torna alla cartella genitore
cd ~                       # torna alla home
cd /                       # vai alla radice
mkdir nuovacartella        # crea una cartella
touch nuovofile.txt        # crea un file vuoto
cp source.txt dest.txt     # copia un file
mv vecchio.txt nuovo.txt   # rinomina o sposta
rm file.txt                # cancella un file (NON va nel cestino, è definitivo)
rm -r cartella             # cancella una cartella e tutto il contenuto (attento)
cat file.txt               # mostra il contenuto di un file
less file.txt              # mostra il contenuto paginato (q per uscire)
grep "parola" file.txt     # cerca "parola" dentro file.txt
find . -name "*.js"        # trova tutti i file .js a partire dalla cartella corrente
echo "ciao"                # stampa "ciao"
echo $HOME                 # stampa il valore della variabile HOME
```

### Pipe e redirect

**Pipe** (`|`): collega l'output di un comando all'input di un altro.

```bash
cat file.txt | grep "errore"
# Stampa solo le righe di file.txt che contengono "errore"
```

**Redirect** (`>`, `>>`): manda l'output a un file.

```bash
ls > lista.txt   # scrive la lista in lista.txt (sovrascrive se esiste)
ls >> lista.txt  # appende alla fine
```

### Differenze fra sistemi

- **macOS / Linux**: shell di default è Bash o Zsh, sintassi simile.
- **Windows**: shell native sono `cmd.exe` (vecchia) e PowerShell. La sintassi è diversa. **Raccomandazione forte**: su Windows installa **WSL2** (Windows Subsystem for Linux), che ti dà un Linux dentro Windows e ti permette di usare la stessa shell della maggior parte degli sviluppatori. Vedi Parte 2.1.

### Dove approfondire

- **Linux command line for beginners** (Ubuntu): https://ubuntu.com/tutorials/command-line-for-beginners
- **Learn Enough Command Line**: https://www.learnenough.com/command-line-tutorial
- Pratica: prova ogni comando della tabella sopra in una cartella di test.

## 1.2 — Version control e Git

### Cos'è il version control

Un sistema che memorizza la storia delle modifiche ai tuoi file. Invece di avere `progetto-finale-v3-DEFINITIVO-davvero.zip`, hai un'unica copia con la storia completa di ogni modifica, chi l'ha fatta, quando, e perché.

Senza version control, lavorare professionalmente è impossibile. Lo userai per:

- Tornare indietro quando qualcosa si rompe.
- Lavorare su più feature in parallelo senza incasinarti.
- Sapere chi (o "te di quando") ha modificato cosa.
- Sincronizzare il progetto fra più macchine.
- Backup automatico (se il repo è su un server come GitHub).

### Cos'è Git

Il sistema di version control distribuito più diffuso al mondo. Praticamente standard.

"Distribuito" significa che ogni copia del repository è completa: contiene tutta la storia, non solo l'ultima versione. Puoi lavorare offline e sincronizzare dopo.

Git è stato creato da Linus Torvalds (sì, quello di Linux) nel 2005 perché aveva bisogno di gestire i contributi al kernel Linux. Nato per gestire migliaia di sviluppatori, funziona benissimo anche per uno solo.

### Concetti fondamentali

**Repository (repo)**: la cartella del tuo progetto sotto version control. Contiene una sotto-cartella nascosta `.git/` con tutta la storia.

**Working directory**: i file che vedi nella cartella, lo stato corrente.

**Staging area** (o "index"): un'area intermedia dove "prepari" le modifiche da salvare nel prossimo commit. Ti permette di scegliere *quali* modifiche includere in un commit (puoi avere modificato 10 file ma committarne solo 3).

**Commit**: uno snapshot delle modifiche stagged. Ha un messaggio (descrizione di cosa hai cambiato), un autore, una data, e un identificatore univoco (un hash SHA-1 lungo, di solito si usano i primi 7 caratteri).

**Branch**: una linea di sviluppo. `main` (in passato `master`) è il branch principale. Puoi creare branch separati per lavorare su feature isolate, poi unire le modifiche al main.

**Merge**: unire le modifiche di un branch in un altro.

**Remote**: una copia del repository su un server (es. GitHub). Lo chiami "remote" perché è remoto rispetto alla tua copia locale. Il remote di default si chiama `origin`.

**Push**: invia i tuoi commit locali al remote.

**Pull**: scarica i commit dal remote nella tua copia locale (è `fetch` + `merge`).

**Clone**: crea una copia locale di un repo remoto.

### Il flusso base

```
File modificato → git add → Staging area → git commit → Repo locale → git push → Repo remoto
```

### Comandi base

```bash
# Inizializzare un repo nuovo
git init

# Clonare un repo esistente
git clone https://github.com/user/repo.git

# Vedere lo stato attuale
git status

# Aggiungere file allo staging
git add file.txt          # un file specifico
git add .                 # tutti i file modificati nella cartella corrente
git add -p                # modalità interattiva, sceglie pezzo per pezzo

# Creare un commit
git commit -m "Messaggio del commit"

# Vedere la storia
git log
git log --oneline         # versione compatta
git log --graph --oneline --all   # con visualizzazione grafica dei branch

# Lavorare con i branch
git branch                # lista i branch
git branch nuovo-branch   # crea un branch
git checkout nuovo-branch # cambia branch
git checkout -b nuovo     # crea e cambia in una sola riga (più comune)
git switch nuovo          # alternativa moderna a checkout

# Sincronizzare col remote
git remote -v             # vedi i remote configurati
git fetch                 # scarica gli aggiornamenti senza applicarli
git pull                  # scarica e applica
git push                  # invia i tuoi commit al remote
git push origin nuovo     # invia il branch "nuovo" al remote "origin"

# Confronti
git diff                  # cosa ho modificato e non ancora aggiunto allo staging
git diff --staged         # cosa è in staging e non ancora committato

# Annullare cose
git restore file.txt      # scarta le modifiche non staged a un file
git reset HEAD file.txt   # rimuove un file dallo staging (le modifiche restano)
git reset --hard          # ATTENZIONE: scarta tutte le modifiche locali
git revert <hash>         # crea un nuovo commit che annulla un commit precedente
```

### Cosa NON committare mai

- File di sistema (`.DS_Store`, `Thumbs.db`).
- Cartelle di dipendenze (`node_modules/`, `venv/`, `__pycache__/`). Sono ricreabili, sono enormi.
- File di build (`dist/`, `build/`). Sono generati dal codice.
- File di configurazione personali dell'IDE (`.idea/`, `.vscode/settings.json`).
- **Secret e credenziali** (file `.env`, chiavi API, password). **Mai. Mai mai mai.** Vedi 1.10.

Per questo serve il file `.gitignore`: dice a Git quali file ignorare. Ne parleremo in dettaglio in Parte 3.

### Branch: il concetto in profondità

Immagina la storia del tuo progetto come un albero. `main` è il tronco. Ogni branch è un ramo. I commit sono nodi su questi rami.

```
main:    A --- B --- C --- D
                 \
feature/x:        E --- F
```

Hai fatto i commit A, B sul main. Hai creato un branch `feature/x` da B. Hai fatto i commit E, F su `feature/x`. Nel frattempo hai fatto C, D su main. Quando vorrai unire `feature/x` su main, dovrai fare un **merge**.

Il merge crea un nuovo commit di "unione" che combina i due branch. Esistono varianti:

- **Merge commit** classico: commit speciale con due genitori. La storia conserva la diramazione.
- **Squash merge**: tutti i commit del branch diventano un singolo commit su main. La storia di main è lineare.
- **Rebase**: prendi i commit del tuo branch e li riapplichi sopra l'ultimo commit di main. La storia diventa lineare, come se avessi sempre lavorato dopo gli altri.

Per uno sviluppatore solo, **squash merge** è il default raccomandato: storia di main pulita, ogni PR è un commit.

### Conflitti di merge

Se due branch modificano la stessa riga dello stesso file, Git non sa quale scegliere e crea un **conflict**. Devi risolverlo a mano: apri il file, vedi i marker `<<<<<<<`, `=======`, `>>>>>>>`, decidi quale versione tenere (o combinarle), salvi, fai `git add` e completi il merge.

Conflitti sono inevitabili. Imparare a risolverli con calma è una skill base.

### Dove approfondire

- **Pro Git Book** (gratuito, completo): https://git-scm.com/book/en/v2
- **Learn Git Branching** (interattivo, visualizzato): https://learngitbranching.js.org/
- **Atlassian Git tutorials**: https://www.atlassian.com/git/tutorials

Investi 4-6 ore qui prima di proseguire. Capire Git solidamente è il singolo investimento più importante della tua carriera.

## 1.3 — GitHub

### Cos'è

GitHub è un servizio cloud che ospita repository Git. Aggiunge funzionalità sopra Git: interfaccia web, gestione utenti, issue tracking, pull request, CI/CD integrato (GitHub Actions), package registry, e altro.

Non è l'unico (esistono GitLab, Bitbucket, Gitea), ma è il più diffuso. Lo useremo come default.

### A cosa serve nel tuo workflow

- **Backup automatico**: ogni `git push` salva il tuo lavoro su un server.
- **Sincronizzazione**: se passi a un'altra macchina, `git clone` ti riporta tutto.
- **Issue tracker**: dove tieni il backlog di cosa devi fare.
- **Pull Request**: meccanismo per proporre modifiche a un branch, con discussione, review, e merge controllato.
- **CI/CD**: GitHub Actions esegue automaticamente test, build, deploy su ogni push o PR.
- **Releases**: pubblicazione strutturata di versioni del software.
- **Container registry**: GHCR per ospitare immagini Docker.
- **Pages**: hosting statico gratuito per siti.

### Account, profilo, sicurezza

L'account GitHub è la tua identità professionale. Curalo:

- Username: scegli un nome professionale. Lo userai negli URL del repo, nei commit, nel profilo. Cambiarlo dopo è doloroso.
- 2FA (autenticazione a due fattori): **obbligatoria**. Usa un'app TOTP (Authy, Aegis, 1Password) o una chiave hardware (Yubikey). NON SMS.
- Email: configura un'email principale, e considera di abilitare "Keep my email addresses private" per non esporre l'email nei commit.

### SSH key

Per spingere codice su GitHub senza inserire password ogni volta, usi una chiave SSH:

1. Generi una coppia di chiavi (pubblica + privata) sul tuo computer.
2. Carichi la chiave pubblica su GitHub.
3. Da quel momento, GitHub ti riconosce automaticamente.

La chiave privata **non lascia mai il tuo computer**. La pubblica può essere condivisa.

I comandi precisi sono in Parte 2.4.

### Repository: pubblico vs privato

- **Pubblico**: chiunque può vedere il codice. Buono per progetti open source.
- **Privato**: solo tu (e chi inviti) può vederlo. Buono per progetti commerciali o personali.

Default raccomandato: **privato** all'inizio. Apri pubblico solo quando sai cosa stai mostrando.

### Issue

Una **issue** è un ticket: descrive un bug, una feature da fare, una domanda. Ha:

- Titolo e descrizione.
- Etichette (`bug`, `feat`, `question`, ecc.).
- Assignee.
- Stato (open / closed).
- Discussione in commenti.

Anche da solo, **apri sempre una issue prima di iniziare a lavorare a una feature**. Diventa la storia di "cosa ho fatto e perché".

### Pull Request (PR)

Una **pull request** è una richiesta di mergiare un branch in un altro. Su GitHub diventa una pagina con:

- Diff completo delle modifiche.
- Discussione e review.
- Status di CI (passa / non passa).
- Pulsante di merge.

Anche da solo, **apri PR invece di mergiare direttamente su main**. Avere lo storico delle PR è oro per ricostruire decisioni mesi dopo.

### GitHub Actions (cenni)

GitHub Actions è un sistema di **CI/CD** (vedi 1.8) integrato nel repo. Definisci dei "workflow" in file YAML dentro `.github/workflows/`. Ogni workflow specifica:

- **Quando** eseguire (on push, on PR, on schedule, ecc.).
- **Cosa** eseguire (job, ognuno con step di comandi).

Esempio concettuale: "ogni volta che apri una PR, esegui i test e il lint". Se i test falliscono, il pulsante di merge si blocca.

### Dove approfondire

- **GitHub Skills** (corsi interattivi): https://skills.github.com/
- **GitHub Docs**: https://docs.github.com/

## 1.4 — Linguaggi di programmazione, runtime, package manager

### Linguaggio compilato vs interpretato

Distinzione un po' sfocata ai giorni nostri, ma utile:

- **Compilato**: il tuo codice viene tradotto in un eseguibile binario una volta sola, prima di girare. Esempi: C, C++, Go, Rust.
- **Interpretato**: il tuo codice viene letto ed eseguito riga per riga (o quasi) ogni volta che lanci il programma, da un altro programma chiamato "interprete" o "runtime". Esempi: Python, JavaScript (Node), Ruby.

In pratica oggi quasi tutto è ibrido (Java compila in bytecode che gira su JVM, Python compila in bytecode in cache, V8 di Node compila JS al volo, ecc.). Il punto pratico per te: alcuni linguaggi hanno bisogno di un **runtime** installato sulla macchina dove gira il codice.

### Runtime

Il runtime è il programma che esegue il tuo codice. Esempi:

- **Node.js** è il runtime per JavaScript fuori dal browser.
- **Python** (l'interprete) è il runtime per Python.
- **JVM** è il runtime per Java/Kotlin/Scala.

Quando "deploy" il tuo progetto, devi assicurarti che il server abbia il runtime giusto. Docker (1.5) risolve questo problema.

### Package manager

Un **package manager** è un tool che gestisce le dipendenze (librerie esterne) del tuo progetto. Esempi:

- **npm** o **yarn** o **pnpm** per Node/JavaScript.
- **pip** o **uv** o **poetry** per Python.
- **cargo** per Rust.
- **go modules** per Go.
- **bundler** per Ruby.

Cosa fa un package manager:

- Installa librerie da un registry (npm registry, PyPI, crates.io, ecc.).
- Mantiene un file di "dipendenze dichiarate" (`package.json`, `requirements.txt`, `Cargo.toml`, ecc.).
- Mantiene un **lock file** (`package-lock.json`, `poetry.lock`, ecc.) che fissa le versioni esatte (anche transitive) per riproducibilità.
- Gestisce script (`npm run dev`, `npm test`).

### Lock file: perché è importante

Senza lock file, oggi installi `lodash@^4.17.0` e ottieni `4.17.21`. Domani, qualcuno (o tu su un'altra macchina) lo installa e ottiene `4.17.22`, perché nel frattempo è uscita. Stesse dipendenze, codice leggermente diverso. Se la versione nuova ha un bug, il tuo software si rompe in modo non riproducibile.

Il lock file fissa esattamente `4.17.21` ovunque. **Sempre committare il lock file.**

### Version manager

Diversi progetti possono richiedere versioni diverse del runtime. Tu lavori al progetto A che vuole Node 18 e al progetto B che vuole Node 20. Come fai?

Un **version manager** ti permette di avere multiple versioni del runtime installate e di switchare automaticamente in base al progetto. Per Node: `nvm`, `fnm`, `volta`. Per Python: `pyenv`, `uv`. Multi-runtime: `mise`, `asdf`.

Il version manager legge un file nel progetto (`.nvmrc`, `.python-version`, `.tool-versions`) e attiva la versione giusta automaticamente quando entri nella cartella.

### Dove approfondire

Dipende dal tuo stack. Se stai imparando JavaScript/Node:

- **MDN Web Docs**: https://developer.mozilla.org/
- **Node.js docs**: https://nodejs.org/docs/

Se Python: https://docs.python.org/3/tutorial/

## 1.5 — Docker e contenitori

### Il problema che Docker risolve

"Funziona sulla mia macchina" è la frase più ripetuta nello sviluppo, e una delle più frustranti. Cause:

- Versione runtime diversa fra dev e produzione.
- Dipendenze di sistema (librerie OS, fonts, certificati) presenti sul tuo PC ma non sul server.
- Configurazioni implicite (variabili d'ambiente, percorsi).

Docker risolve standardizzando: pacchetti il tuo progetto + tutto ciò di cui ha bisogno (runtime, librerie, file) in un'**immagine**, e questa immagine gira identica ovunque.

### VM vs container

Una **virtual machine (VM)** simula un intero computer: hardware virtuale, sistema operativo completo, kernel, tutto. Pesante: GB di disco, secondi-minuti per avviarsi.

Un **container** condivide il kernel del sistema host. Pacchetta solo l'applicazione e le sue dipendenze user-space. Leggero: MB di disco, secondi per avviarsi.

Docker usa container.

### Concetti chiave

**Immagine (image)**: una "fotografia" del filesystem e configurazione necessari per eseguire un'applicazione. È **immutabile** e a strati (ogni istruzione del Dockerfile crea uno strato sovrapposto al precedente). Si identifica con un nome e un tag (es. `node:20-alpine`).

**Container**: un'istanza in esecuzione di un'immagine. È **mutabile** (ma le modifiche dentro un container scompaiono quando lo termini, a meno che non monti volumi). Puoi avviare 100 container dalla stessa immagine.

**Dockerfile**: il "ricettario" testuale che descrive come costruire un'immagine. Contiene istruzioni come `FROM`, `RUN`, `COPY`, `CMD`.

**Registry**: un server che ospita immagini Docker. Docker Hub è quello pubblico più famoso. GHCR (GitHub Container Registry) è quello integrato con GitHub.

**Docker Compose**: un tool per orchestrare più container insieme. Definisci in `docker-compose.yml` i servizi del tuo progetto (app + database + cache + ecc.) e li lanci tutti insieme con `docker compose up`.

**Volume**: un meccanismo per persistere dati al di fuori del container. Senza volumi, i dati nel container scompaiono quando lo distruggi.

**Network**: rete virtuale che collega i container fra loro.

### Esempio mentale

Un progetto Node.js che usa Postgres come database:

```
[Container app: Node 20 + tuo codice]  ←→  [Container db: Postgres 16]
                  |
            [Volume per dati DB]
```

Senza Docker: dovresti installare Node 20 e Postgres 16 sulla tua macchina, configurarli, ricordarti come ricominciare se cambi macchina, sperare che il server abbia gli stessi setup.

Con Docker: scrivi un `docker-compose.yml`, lanci `docker compose up`, hai entrambi i servizi che parlano fra loro. Il setup è descritto nel file, riproducibile ovunque.

### Quando usarlo

Sempre, se il progetto va in produzione. Per script throwaway o esperimenti, è eccessivo.

### Comandi base che imparerai

```bash
docker build -t mio-progetto .       # costruisci un'immagine
docker run -p 3000:3000 mio-progetto # esegui un container
docker ps                            # lista container in esecuzione
docker images                        # lista immagini locali
docker logs <container>              # vedi i log di un container
docker exec -it <container> sh       # apri una shell dentro un container
docker compose up -d                 # lancia tutti i servizi (modalità "detached")
docker compose down                  # spegne tutti i servizi
docker compose logs -f               # segui i log
```

### Dove approfondire

- **Docker official docs / Get Started**: https://docs.docker.com/get-started/
- **Docker for beginners**: https://docker-curriculum.com/

## 1.6 — Editor / IDE

### Cos'è

Un **editor** è un programma per scrivere codice. Un **IDE** (Integrated Development Environment) è un editor con dentro tool integrati: debugger, terminale, controllo versione, ecc.

### VS Code (raccomandato)

[Visual Studio Code](https://code.visualstudio.com/) è gratuito, multipiattaforma, estensibile, e ha l'ecosistema di estensioni più ricco. Default per la maggior parte degli sviluppatori web.

Alternative valide: **JetBrains IDEs** (IntelliJ, PyCharm, WebStorm) — a pagamento, eccellenti se lavori molto con un linguaggio specifico. **Neovim** — terminale, ripida curva di apprendimento, ma estremamente potente per chi ci si dedica.

### Estensioni essenziali per VS Code

- **Linguaggio del tuo stack** (TypeScript, Python, Go, ecc.).
- **GitLens** — vista avanzata di Git inline.
- **ESLint / Prettier** (per JS/TS) — lint e formatting.
- **Docker** — gestione container.
- **EditorConfig** — uniforma stile fra editor.
- **Error Lens** — mostra errori inline.
- **Code Spell Checker** — controlla refusi anche nel codice.

### Dove approfondire

- **VS Code docs / Getting Started**: https://code.visualstudio.com/docs

## 1.7 — Concetti di sviluppo software

### Test

Il codice di test è codice che verifica che il tuo codice "vero" funzioni.

Tipologie:

- **Unit test**: testano una singola funzione/classe in isolamento. Veloci, tanti. Es. "la funzione `somma(2, 3)` restituisce 5".
- **Integration test**: testano l'interazione fra più componenti (es. controller + database). Più lenti, meno numerosi.
- **End-to-end test (e2e)**: testano l'intero flusso utente, di solito attraverso un browser o un client HTTP. Lenti, pochi.
- **Smoke test**: pochi test che verificano che le feature critiche non siano completamente rotte. Si eseguono dopo il deploy.

### Perché si fanno

- Trovare bug prima degli utenti.
- Avere il coraggio di refactor: se i test passano, sai di non aver rotto comportamenti.
- Documentare il comportamento atteso del codice (i test sono specifiche eseguibili).
- Forzarti a pensare al design: codice difficile da testare di solito è codice mal disegnato.

### TDD (Test-Driven Development)

Una tecnica dove **scrivi il test prima dell'implementazione**:

1. Scrivi un test che fallisce (perché la feature non esiste).
2. Scrivi il minimo codice che fa passare il test.
3. Refactor: pulisci il codice, i test devono restare verdi.
4. Ripeti.

Vantaggi: test sempre presenti, design guidato dai test, ti forza a pensare al "cosa" prima del "come".

### Coverage

Percentuale di righe (o branch, o funzioni) di codice eseguite dai test. Strumenti come `nyc`/`c8` (Node), `coverage.py` (Python), `go test -cover` (Go) la calcolano.

**Trappola**: 100% coverage non significa "codice perfetto". Significa che ogni riga è stata eseguita, non che ogni caso è stato testato. Un test che esegue una riga senza assertion è inutile ma conta.

Coverage utile come segnale negativo (sotto X% sicuramente mancano test) ma non come obiettivo per sé.

### Linting

Un **linter** analizza il codice senza eseguirlo, cercando:

- Errori di sintassi.
- Anti-pattern (codice che funziona ma è considerato cattiva pratica).
- Inconsistenze stilistiche.

Esempi: ESLint (JS/TS), Pylint/Ruff (Python), golangci-lint (Go), Clippy (Rust).

### Formatting

Un **formatter** riformatta automaticamente il codice secondo regole stilistiche (indentazione, spaziatura, line length, ecc.).

Esempi: Prettier (multi-linguaggio), Black (Python), gofmt (Go), rustfmt (Rust).

Differenza con linting: il formatter non discute, riformatta. Il linter segnala.

### Type checking

In linguaggi tipizzati staticamente (TypeScript, Go, Rust, Java), il compilatore controlla i tipi a tempo di compilazione. Errori di tipo li trovi prima di eseguire.

In linguaggi dinamici (Python, JavaScript), c'è il **type checking opzionale** (mypy in Python, TypeScript per JS). Aggiunge un layer di sicurezza ma richiede annotazioni.

### Dove approfondire

- **Test-Driven Development by Example** (Kent Beck) — il libro che ha definito TDD.
- **The Art of Unit Testing** (Roy Osherove) — guida pratica.

## 1.8 — Concetti DevOps

### CI (Continuous Integration)

Pratica di integrare frequentemente il codice (più volte al giorno) e verificare automaticamente che tutto funzioni.

In pratica: ogni volta che pushi codice (o apri una PR), un sistema automatico (es. GitHub Actions) esegue:

- Build del progetto.
- Test.
- Lint.
- Type check.
- Security scan.

Se qualcosa fallisce, blocchi il merge.

### CD (Continuous Delivery / Continuous Deployment)

- **Continuous Delivery**: il codice è sempre pronto per essere deployato, ma il deploy lo fai a mano.
- **Continuous Deployment**: ogni commit che passa CI viene deployato automaticamente in produzione.

Per progetti solo, di solito si usa Continuous Delivery: deploy manuale dopo CI verde.

### Pipeline

L'insieme dei passi automatizzati che il tuo codice attraversa: build → test → security scan → push immagine → deploy. Definita in YAML (GitHub Actions, GitLab CI) o DSL (Jenkins).

### Deploy

Il processo di portare il tuo codice in un ambiente dove gira (staging, production). Strategie:

- **Recreate**: spegni il vecchio, accendi il nuovo. Downtime breve.
- **Rolling**: sostituzione graduale delle istanze. Zero downtime se hai >1 istanza.
- **Blue-Green**: due ambienti identici, switchi il traffico fra l'uno e l'altro. Rollback istantaneo.
- **Canary**: deploy graduale al 5%, 25%, 50%, 100% del traffico. Per feature ad alto rischio.

### Rollback

Tornare alla versione precedente quando la nuova ha problemi. Se la procedura non è documentata e testata, non funzionerà nel momento in cui ti serve davvero.

### Observability (osservabilità)

La capacità di capire cosa sta facendo il tuo sistema in produzione. Tre pilastri:

- **Logs**: cosa è successo (eventi testuali con timestamp, livello, messaggio).
- **Metrics**: quanto/quanti (numeri nel tempo: RPS, error rate, latency, CPU).
- **Traces**: dove e quanto è durato (mappe di chiamate fra componenti distribuiti).

Per un progetto piccolo, logs strutturati + metrics base bastano.

### Health check

Endpoint che il tuo servizio espone (di solito `/health`) e che restituisce un OK se il servizio è funzionante. Usato da load balancer, monitoring, Docker (`HEALTHCHECK`), Kubernetes.

### Alert

Notifiche automatiche quando qualcosa va male: error rate troppo alto, health check rosso, disco pieno, certificato in scadenza. Email, Slack, Telegram, PagerDuty.

### Dove approfondire

- **The DevOps Handbook** (Gene Kim et al.) — bibbia del DevOps.
- **Site Reliability Engineering** (Google, gratis online): https://sre.google/books/

## 1.9 — Documentazione e convenzioni

### README

Il primo file che chiunque legge entrando in un repo. Struttura standard:

- Titolo e descrizione una riga.
- Quick start: come eseguire il progetto in locale in 3-5 comandi.
- Stack tecnologico.
- Link a documentazione più dettagliata in `docs/`.
- License.

### CHANGELOG

Lista delle modifiche per versione. Formato standard: [Keep a Changelog](https://keepachangelog.com/).

```markdown
## [Unreleased]

### Added
- Endpoint /api/notes per creare note.

## [0.2.0] - 2026-05-15

### Added
- Autenticazione OAuth2.

### Fixed
- Bug nel parsing date.
```

### ADR (Architecture Decision Records)

Brevi documenti che catturano una decisione architetturale, le opzioni considerate, e le conseguenze. Vivono in `docs/adr/`. Servono al "te" futuro per ricordare *perché* hai fatto una certa scelta.

Template:

```markdown
# ADR 0001: Scelta del database

Stato: Accettato
Data: 2026-05-06

## Contesto
Stiamo costruendo un'API per gestire note personali. Serve un DB per persistere dati strutturati. Volume previsto: < 1M righe.

## Opzioni considerate
- A: PostgreSQL — relazionale, maturo, query complesse.
- B: SQLite — file locale, zero setup, single-user.
- C: MongoDB — document store, schema flessibile.

## Decisione
PostgreSQL. Motivo: relazioni fra note, tag, e utenti sono naturali in modello relazionale. SQLite escluso perché in futuro vogliamo multi-user.

## Conseguenze
- Setup più complesso (richiede container Postgres in dev).
- Permette query SQL ricche.
- Lock-in moderato a Postgres.

## Condizioni di rivalutazione
Se il progetto resta single-user oltre 12 mesi, riconsiderare SQLite per semplicità.
```

### Conventional Commits

Formato standardizzato per i messaggi di commit:

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

Type possibili:

- `feat`: nuova funzionalità.
- `fix`: bug fix.
- `docs`: solo documentazione.
- `style`: formatting (no code change).
- `refactor`: refactor senza cambio comportamento.
- `perf`: miglioramento performance.
- `test`: aggiunta/modifica test.
- `chore`: manutenzione, deps, tooling.
- `ci`: modifiche CI.
- `build`: modifiche build system.

Esempi:

```
feat(auth): add OAuth2 login flow
fix(api): handle null user in notes endpoint
docs: update README with deploy instructions
chore(deps): bump express from 4.18.2 to 4.19.0
```

Breaking change: `feat!:` con `!` o footer `BREAKING CHANGE: ...`.

Vantaggio: macchine possono parsare i commit per generare changelog automaticamente, decidere il bump di versione semver, ecc.

### Semantic Versioning (SemVer)

Schema di numerazione versioni: `MAJOR.MINOR.PATCH`.

- **MAJOR**: cambia quando rompi la compatibilità con la versione precedente (breaking change).
- **MINOR**: cambia quando aggiungi funzionalità in modo backward-compatible.
- **PATCH**: cambia quando fai bug fix backward-compatible.

Esempio: passi da `1.4.2` a `1.4.3` per un bug fix, a `1.5.0` per una feature, a `2.0.0` per una rimozione di endpoint API.

Pre-release: `1.0.0-rc.1`, `1.0.0-beta.2`.

Pre-1.0: `0.x.y`. La regola "minor = backward compatible" non si applica. Quando l'API è stabile, esci con `1.0.0`.

### Dove approfondire

- **Conventional Commits**: https://www.conventionalcommits.org/
- **Semantic Versioning**: https://semver.org/
- **Keep a Changelog**: https://keepachangelog.com/

## 1.10 — Sicurezza base

### Secret e perché non si committano

Un **secret** è qualsiasi valore che dà accesso a qualcosa di tuo: password, API key, JWT secret, certificato privato, token OAuth, stringa di connessione DB.

Se committi un secret in un repo Git:

- Anche se cancelli il file dopo, il secret resta nella storia di Git.
- Se il repo è pubblico, è esposto al mondo. Bot scansionano GitHub costantemente per trovarne.
- Anche su repo privati, è esposto a chi ha accesso (anche solo in lettura).
- I provider come AWS, Stripe rilevano i loro secret pubblicati e ti scrivono per dirti di ruotarli, perché succede continuamente.

**Regola assoluta: i secret non vivono nei file del progetto.** Vivono in:

- File `.env` locali, **ignorati da Git**.
- Variabili d'ambiente del server di produzione.
- Secret manager (1Password CLI, AWS Secrets Manager, Doppler, Vault).

Vedi Parte 3.6 per setup pratico.

### HTTPS, SSL/TLS, certificati

**HTTP** trasmette dati in chiaro: chiunque sniffi la rete vede password, cookie, contenuti.

**HTTPS** cifra il traffico fra browser e server con **TLS** (Transport Layer Security, evoluzione di SSL).

Per servire HTTPS, il server ha bisogno di un **certificato** firmato da una **Certificate Authority (CA)** riconosciuta. Browser si fidano dei certificati firmati da CA note (Let's Encrypt, DigiCert, ecc.).

**Let's Encrypt** è una CA gratuita e automatizzabile. Default per tutti i progetti web. Lo configuri (di solito via Caddy, certbot, o il tuo cloud) e si rinnova da solo.

### Autenticazione vs autorizzazione

- **Autenticazione (authn)**: chi sei? (login con username + password, OAuth, ecc.)
- **Autorizzazione (authz)**: cosa puoi fare? (sei ammin? puoi vedere questo dato?)

Sono concetti distinti. Spesso vengono confusi.

### OWASP Top 10

Lista delle 10 vulnerabilità web più diffuse, mantenuta da OWASP. Aggiornata ogni qualche anno. Da conoscere:

- **Injection** (SQL injection, command injection): input utente eseguito come codice.
- **Broken Authentication**: login fragili.
- **Sensitive Data Exposure**: dati sensibili in chiaro o non protetti.
- **Broken Access Control**: utente A può vedere dati di utente B.
- **Security Misconfiguration**: default insicuri.
- **Cross-Site Scripting (XSS)**: codice JavaScript malevolo iniettato.
- **Insecure Deserialization**.
- **Using Components with Known Vulnerabilities**: dipendenze con CVE note.
- **Insufficient Logging & Monitoring**.

Risorsa: https://owasp.org/Top10/

### Vulnerability scanning

Strumenti automatici che scansionano:

- **Codice tuo** (SAST — Static Application Security Testing): Semgrep, SonarQube.
- **Dipendenze** (SCA — Software Composition Analysis): `npm audit`, `pip-audit`, Snyk, Dependabot.
- **Immagini Docker**: Trivy, Grype.
- **Secret nel repo**: gitleaks, trufflehog.
- **Applicazione runtime** (DAST — Dynamic Application Security Testing): OWASP ZAP, Burp Suite.

Da integrare in CI per scansione automatica.

### Dove approfondire

- **OWASP**: https://owasp.org/
- **Web Security Academy** (PortSwigger, gratis): https://portswigger.net/web-security

---

# Parte 2 — Setup della tua macchina

Questa parte è pratica: aprirai il terminale e installerai gli strumenti che userai per il resto della guida.

## 2.1 — Sistema operativo

### macOS

Pronto all'uso. La maggior parte degli sviluppatori usa macOS.

### Linux

Pronto all'uso (Ubuntu, Fedora, Arch, Debian sono tutte buone scelte). Eccellente per sviluppo.

### Windows

**Raccomandazione forte**: installa **WSL2** (Windows Subsystem for Linux 2). È un Linux completo dentro Windows, integrato col filesystem e con VS Code.

Procedura:

1. Apri PowerShell come amministratore.
2. Esegui:
   ```
   wsl --install
   ```
3. Riavvia.
4. Si installa Ubuntu di default. Configura username e password Linux.
5. Da quel momento, apri "Ubuntu" da Start e hai un terminale Linux. **Tutta questa guida assume Linux/macOS dentro il terminale.** Se sei su Windows nativo (cmd/PowerShell), molti comandi cambieranno: passa a WSL2.
6. È buona norma sincronizzare i repository e aggiornare i pacchetti preinstallati:
   ```
   sudo apt update && sudo apt upgrade -y
   ```

Riferimento: https://learn.microsoft.com/en-us/windows/wsl/install

## 2.2 — Terminale migliore

Il terminale di default è funzionale ma ne esistono di più piacevoli:

- **macOS**: [iTerm2](https://iterm2.com/).
- **Windows**: Windows Terminal (gratuito su Microsoft Store).
- **Linux**: Alacritty, Kitty, o quello del tuo desktop environment.

Configura un font monospaziato decente (es. JetBrains Mono, Fira Code, MonoLisa) e un tema scuro che non ti faccia bruciare gli occhi.

## 2.3 — Editor: VS Code

### Installazione

Scarica da https://code.visualstudio.com/ e installa.

### Configurazione iniziale

Apri VS Code, premi `Ctrl+Shift+P` (`Cmd+Shift+P` su Mac), apri le impostazioni JSON ("Preferences: Open User Settings (JSON)"). Aggiungi:

```json
{
  "editor.formatOnSave": true,
  "editor.tabSize": 2,
  "editor.insertSpaces": true,
  "editor.rulers": [80, 120],
  "files.trimTrailingWhitespace": true,
  "files.insertFinalNewline": true,
  "files.eol": "\n",
  "git.autofetch": true,
  "git.confirmSync": false,
  "terminal.integrated.fontFamily": "JetBrains Mono"
}
```

Controlla di avere il font "JetBrains Mono" **Impostazioni > Caratteri**, 
altrimenti scarica il file zip dal [sito ufficiale JetBrains Mono](https://jetbrains.com) 
estrai, vai in font/tff e installa almeno questi font:

1. **`JetBrainsMono-Regular.ttf`** (Fondamentale)
2. **`JetBrainsMono-Bold.ttf`**
3. **`JetBrainsMono-Italic.ttf`**

**Procedura:** Selezionali → Tasto destro → **Installa**. 
*Nota: Evita le versioni "NL" per mantenere le legature.*

### Estensioni

Apri il pannello Extensions (`Ctrl+Shift+X`) e installa:

- **GitLens** — vista Git avanzata.
- **EditorConfig for VS Code**.
- **Error Lens**.
- **Code Spell Checker**.
- **Prettier - Code formatter** (se lavori con JS/TS).
- **ESLint** (se lavori con JS/TS).
- **Python** (se lavori con Python).
- **Docker**.
- **YAML**.
- **GitHub Pull Requests and Issues**.

## 2.4 — Git

### Installazione

- **macOS**: `brew install git` (installa prima [Homebrew](https://brew.sh/) se non l'hai).
- **Linux Ubuntu**: `sudo apt update && sudo apt install git`.
- **Windows**: già incluso in WSL2 Ubuntu, oppure scarica https://git-scm.com/download/win.

Verifica:

```bash
git --version
```

Deve restituire `git version 2.x.x` (almeno 2.30+).

### Configurazione iniziale

Esegui (sostituisci con i tuoi dati):

```bash
git config --global user.name "Mario Rossi"
git config --global user.email "mario@example.com"
git config --global init.defaultBranch main
git config --global pull.rebase false
git config --global core.autocrlf input    # Linux/macOS
# git config --global core.autocrlf true   # Windows nativo (no WSL)
git config --global core.editor "code --wait"
```

Verifica:

```bash
git config --list
```

### SSH key per GitHub

Genera una nuova chiave SSH:

```bash
ssh-keygen -t ed25519 -C "mario@example.com"
```

Premi Enter per accettare la posizione di default (`~/.ssh/id_ed25519`). Imposta una **passphrase** (consigliato — è una password aggiuntiva sulla chiave privata).

Avvia l'agent SSH e aggiungi la chiave:

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

Su macOS, per ricordare la passphrase nel keychain:

```bash
ssh-add --apple-use-keychain ~/.ssh/id_ed25519
```

Copia la **chiave pubblica** (NON la privata!) negli appunti:

```bash
# macOS
cat ~/.ssh/id_ed25519.pub | pbcopy
# Linux (con xclip installato)
cat ~/.ssh/id_ed25519.pub | xclip -selection clipboard
# WSL2
cat ~/.ssh/id_ed25519.pub | clip.exe
# Sempre funziona: copia a mano dall'output
cat ~/.ssh/id_ed25519.pub
```

Vai su GitHub → Settings → SSH and GPG keys → New SSH key. Incolla, dai un titolo (es. "Laptop Mario 2026"), salva.

Test:

```bash
ssh -T git@github.com
```

Dovresti vedere: `Hi <username>! You've successfully authenticated...`

## 2.5 — Account GitHub

Crea un account su https://github.com/ se non l'hai.

### Sicurezza

- **Username**: scegli con cura, è la tua identità professionale.
- **Email**: principale. Considera "Keep my email addresses private" in Settings → Emails.
- **2FA**: Settings → Password and authentication → Two-factor authentication → abilita con app TOTP (Authy/Aegis/1Password). **Salva i recovery codes** (stampali, mettili in cassaforte).

### Profilo

- Foto profilo professionale (tua faccia o avatar consistente).
- Bio breve.
- Link al tuo sito se ne hai.

### Verifica

Crea un repo di test "hello-world":

```bash
mkdir ~/test-github && cd ~/test-github
git init
echo "# Test" > README.md
git add README.md
git commit -m "chore: initial commit"
git branch -M main
```

Su GitHub → New repository → nome `hello-world`, **private**, NON inizializzare con README/license. Crea.

Aggiungi il remote (sostituisci `<user>`):

```bash
git remote add origin git@github.com:<user>/hello-world.git
git push -u origin main
```

Vai su GitHub e verifica che il file `README.md` sia visibile. Se sì, l'SSH funziona.

Cancella il repo di test alla fine.

## 2.6 — Version manager

Sceglieremo uno per ogni runtime principale. Per questa guida useremo Node.js come stack di esempio. Adatta se usi altro.

### Node: nvm (raccomandato)

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
```

Chiudi e riapri il terminale. Verifica:

```bash
nvm --version
```

Installa Node:

```bash
nvm install --lts          # ultima versione LTS (long-term support)
nvm use --lts
nvm alias default lts/*
node --version             # verifica
npm --version
```

### Python: pyenv (se ti serve Python)

```bash
curl https://pyenv.run | bash
```

Aggiungi al tuo shell config (`.zshrc` o `.bashrc`):

```bash
export PYENV_ROOT="$HOME/.pyenv"
[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"
```

Riapri terminale, installa Python:

```bash
pyenv install 3.12.3
pyenv global 3.12.3
python --version
```

### Multi-runtime: mise (alternativa unica)

Se gestisci più runtime, [`mise`](https://mise.jdx.dev/) li unifica:

```bash
curl https://mise.run | sh
```

Aggiungi al tuo shell config: `eval "$(mise activate bash)"` (o `zsh`).

```bash
mise use --global node@20 python@3.12
```

## 2.7 — Docker

### Installazione

- **macOS**: scarica Docker Desktop da https://www.docker.com/products/docker-desktop/. Avvialo.
- **Windows**: Docker Desktop, ma DEVI avere WSL2 attivo. Si integra con la tua distro Linux.
- **Linux**: Docker Engine direttamente. Segui le istruzioni del tuo distro: https://docs.docker.com/engine/install/

Su Linux, aggiungi il tuo utente al gruppo `docker` per evitare di usare `sudo` ogni volta:

```bash
sudo usermod -aG docker $USER
# Logout e login per applicare
```

### Verifica

```bash
docker --version
docker compose version
docker run --rm hello-world
```

L'ultimo comando scarica ed esegue una mini-immagine di test. Se vedi "Hello from Docker!", funziona.

## 2.8 — Claude Code e plugin

### Installazione Claude Code

Segui le istruzioni ufficiali su https://docs.claude.com/. Tipicamente:

```bash
npm install -g @anthropic-ai/claude-code
```

Esegui per la prima volta:

```bash
claude
```

Ti chiederà di autenticarti. Segui le istruzioni a schermo.

### Plugin everything-claude-code

Plugin che aggiunge slash command e workflow utili:

- Sito: https://github.com/affaan-m/everything-claude-code
- Segui il README per l'installazione (varia con le versioni).

Tipicamente clonarsi sotto `~/.claude/plugins/` o simile, oppure installazione via comando dedicato.

Verifica con:

```
/harness-audit
```

dentro Claude Code. Deve restituire OK.

## 2.9 — Tool aggiuntivi

### gh — GitHub CLI

Permette di interagire con GitHub dal terminale (creare PR, issue, ecc.):

```bash
# macOS
brew install gh
# Ubuntu/Debian
sudo apt install gh
```

Login:

```bash
gh auth login
```

Scegli SSH come metodo (hai già la chiave dalla Parte 2.4).

### jq — JSON manipulator

Per processare JSON in shell:

```bash
brew install jq           # macOS
sudo apt install jq       # Ubuntu
```

### httpie — HTTP client friendly

Più amichevole di `curl` per testare API:

```bash
brew install httpie
sudo apt install httpie
```

### gitleaks — secret scanner

```bash
brew install gitleaks
# Linux: scarica binario da https://github.com/gitleaks/gitleaks/releases
```

### trivy — vulnerability scanner

```bash
brew install trivy
# Linux: vedi https://aquasecurity.github.io/trivy/
```

### lefthook — pre-commit hooks runner

```bash
brew install lefthook
# Linux: vedi https://github.com/evilmartians/lefthook
```

## 2.10 — Verifica completa

Apri un nuovo terminale e verifica che tutto risponda:

```bash
git --version
node --version
npm --version
docker --version
docker compose version
gh --version
jq --version
http --version
gitleaks version
trivy --version
lefthook version
claude --version
ssh -T git@github.com
```

Tutti devono restituire una versione senza errori. L'ultimo deve dirti "Hi <username>".

Se qualcosa non funziona, fermati e risolvi prima di proseguire. Procedere con un setup rotto causa solo confusione.

---

# Parte 3 — Il Tuo Primo Progetto: Setup Completo

Useremo come progetto-esempio una **Personal Notes API**: un servizio HTTP che permette di creare, leggere, modificare, cancellare note testuali. Stack: Node.js + Express + PostgreSQL.

Adatta i comandi al tuo stack se usi altro. La logica è invariata.

Tempo stimato: 3-4 ore la prima volta. Diventa 30 minuti quando avrai esperienza.

## 3.1 — Definire lo scope (Fase 0 spiegata)

Prima di toccare codice, scriviamo cosa stiamo facendo. Sembra eccessivo per un progetto piccolo — non lo è. Ti previene da costruire la cosa sbagliata.

### Crea la cartella del progetto

```bash
mkdir -p ~/dev/personal-notes-api
cd ~/dev/personal-notes-api
```

### Scrivi `docs/scope.md`

Crea la cartella `docs/`:

```bash
mkdir docs
```

Crea `docs/scope.md` con questo contenuto (apri VS Code: `code .`):

```markdown
# Personal Notes API — Scope

## Cosa fa
Un'API HTTP per creare, leggere, modificare e cancellare note testuali personali.
Single-user (per ora). Le note hanno: id, titolo, corpo, tag, timestamp creazione/modifica.

## Cosa NON fa (non-scope)
- Nessuna interfaccia utente. È solo un'API.
- Nessun multi-utente, nessuna autenticazione utente (le richieste sono autenticate
  da un singolo API token).
- Nessuna ricerca full-text avanzata.
- Nessun supporto offline / sync.

## Utenti target
Solo io. La API gira su un server personale e è chiamata da script o futuri client.

## Vincoli
- Tempo: ho ~30 ore per il MVP.
- Budget cloud: < €10/mese.
- Performance: < 200ms p95 latency su CRUD semplici.
- Compliance: nessun dato personale di terzi → no GDPR. Nessuno requisito di accessibilità.

## Definition of Done (MVP)
- CRUD completo via REST API.
- Autenticazione con API token (Bearer header).
- Persistenza in PostgreSQL.
- Tests con coverage >= 70% (>= 100% sui path di auth).
- Deploy funzionante su staging.
- Documentazione API (OpenAPI).
- Security scan PASS.
- Procedura di deploy e rollback documentata in docs/.

## Stima
- Setup progetto e infra: 4 ore (questa parte 3).
- Auth + CRUD base: 8 ore.
- Test e refinement: 6 ore.
- Deploy staging + production: 6 ore.
- Buffer 50%: +12 ore.
- Totale: ~36 ore.

## Decisioni iniziali
Vedi docs/adr/0001-licenza.md, docs/adr/0002-stack.md, docs/adr/0003-database.md.
```

Salva. Hai appena scritto un documento di scope professionale. Anche se sei solo, anche se è un progetto da 30 ore, **questo file ti previene scope creep** (tendenza ad aggiungere feature non pianificate).

### Stima realistica

Notato il "Buffer 50%"? Chiunque sottostima. Moltiplica la prima stima per 1.5 sempre. Se il totale supera il tempo che hai, **taglia feature**, non comprimere stime.

### ADR iniziali

Crea `docs/adr/`:

```bash
mkdir docs/adr
```

Crea `docs/adr/0001-licenza.md`:

```markdown
# ADR 0001: Licenza

Stato: Accettato
Data: 2026-05-06

## Contesto
Devo decidere la licenza prima del primo commit. È un progetto personale che potrei rendere open source in futuro.

## Opzioni considerate
- A: MIT — permissiva, massima adozione.
- B: Apache 2.0 — permissiva con clausola brevetti.
- C: AGPL v3 — copyleft forte, forza derivati open.
- D: Proprietary — chiuso.

## Decisione
MIT. Motivo: progetto personale, non commerciale, voglio massima libertà di riuso anche futuro. Apache 2.0 sarebbe sovradimensionato.

## Conseguenze
- Chiunque può forkare e usare anche in progetti chiusi.
- Niente vincoli contributori.
- Incompatibile con codice GPL: non posso includere librerie GPL.

## Condizioni di rivalutazione
Se diventa un prodotto commerciale, considerare dual licensing (MIT + Commercial).
```

Crea `docs/adr/0002-stack.md`:

```markdown
# ADR 0002: Scelta dello stack

Stato: Accettato
Data: 2026-05-06

## Contesto
Devo scegliere linguaggio e framework per la API.

## Opzioni considerate
- A: Node.js + Express — JavaScript, ecosistema enorme, conoscenza pre-esistente.
- B: Python + FastAPI — async nativo, validazione tipo eccellente con Pydantic.
- C: Go + chi — performance, binari statici, deploy semplice.

## Decisione
Node.js + Express + TypeScript. Motivo: conosco già JavaScript, TypeScript aggiunge type safety, Express è il default per API REST in Node, ecosistema npm gigante per ogni esigenza.

## Conseguenze
- Velocità di sviluppo alta grazie alla conoscenza pre-esistente.
- Ecosistema npm comporta supply chain risk: dipendenze da gestire con attenzione.
- TypeScript aggiunge un layer di build (compilazione).

## Condizioni di rivalutazione
Se le performance diventano critiche (> 1000 RPS), valutare Go.
```

Crea `docs/adr/0003-database.md`:

```markdown
# ADR 0003: Scelta del database

Stato: Accettato
Data: 2026-05-06

## Contesto
Serve un DB per persistere le note.

## Opzioni considerate
- A: PostgreSQL — relazionale, maturo, query SQL ricche.
- B: SQLite — file locale, zero setup, single-user perfetto.
- C: MongoDB — document store.

## Decisione
PostgreSQL. Motivo: anche se ora è single-user, voglio mantenere apertura a multi-user futuro. SQL è skill universale, vale la pena.

## Conseguenze
- Setup più complesso in dev (richiede container Postgres).
- Possibilità di query complesse, vincoli FK, transazioni robuste.
- Hosting più costoso di SQLite (gratis su file).

## Condizioni di rivalutazione
Se il progetto resta strettamente single-user oltre 12 mesi, riconsiderare SQLite per semplicità.
```

Hai ora tre ADR. Sembrano sovradimensionati per un piccolo progetto, ma ti garantisco: tra sei mesi, quando ti chiederai "perché diavolo ho scelto Postgres invece di SQLite?", li rileggerai e ringrazierai.

## 3.2 — Crea il repository GitHub

Vai su GitHub → New repository:

- Owner: tu.
- Repository name: `personal-notes-api`.
- Description: "Personal notes API. Single-user, REST, PostgreSQL."
- Visibility: **Private**.
- **NON spuntare** "Add a README", "Add .gitignore", "Choose a license". Li aggiungeremo noi.

Crea.

GitHub ti mostrerà istruzioni per pushare un repo esistente. Le seguiremo dopo.

## 3.3 — Setup locale del progetto

Sei nella cartella `~/dev/personal-notes-api`. Inizializza Git:

```bash
git init
git branch -m main
```

Crea i file base che ogni progetto deve avere.

### `.gitignore`

Crea `.gitignore`:

```bash
cat > .gitignore << 'EOF'
# Node
node_modules/
npm-debug.log*
yarn-debug.log*
yarn-error.log*

# Build
dist/
build/

# TypeScript
*.tsbuildinfo

# Coverage
coverage/
.nyc_output/

# Secrets
.env
.env.local
.env.*.local
*.pem
*.key
secrets/

# IDE
.idea/
.vscode/*
!.vscode/extensions.json
!.vscode/settings.json.recommended
*.swp
*.swo

# OS
.DS_Store
Thumbs.db

# Logs
*.log
logs/

# Docker
.dockerignore.local

# Misc
.cache/
*.tmp
EOF
```

### `.editorconfig`

```bash
cat > .editorconfig << 'EOF'
root = true

[*]
charset = utf-8
end_of_line = lf
indent_style = space
indent_size = 2
insert_final_newline = true
trim_trailing_whitespace = true

[*.md]
trim_trailing_whitespace = false
EOF
```

### `LICENSE` (MIT)

```bash
cat > LICENSE << 'EOF'
MIT License

Copyright (c) 2026 Mario Rossi

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
EOF
```

(Sostituisci con il tuo nome.)

### `README.md`

```bash
cat > README.md << 'EOF'
# Personal Notes API

Personal notes API. Single-user, REST, PostgreSQL.

## Quick start

Prerequisites: Node 20+, Docker, Docker Compose.

```bash
git clone git@github.com:<user>/personal-notes-api.git
cd personal-notes-api
cp .env.example .env
# Edit .env with your values
docker compose up -d
npm install
npm run migrate
npm run dev
```

API will be running on http://localhost:3000.

## Stack

- Node.js 20 + TypeScript
- Express
- PostgreSQL 16
- Vitest (testing)
- Docker + Docker Compose

## Documentation

- [Scope](docs/scope.md)
- [Architecture Decision Records](docs/adr/)
- [Deploy procedure](docs/deploy.md)
- [Rollback procedure](docs/rollback.md)
- [Runbook](docs/runbook.md)

## License

MIT — see [LICENSE](LICENSE).
EOF
```

### `CHANGELOG.md`

```bash
cat > CHANGELOG.md << 'EOF'
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Initial project scaffolding.
EOF
```

### Inizializzazione Node

```bash
echo "20" > .nvmrc
nvm use
npm init -y
```

Apri `package.json` (`code package.json`) e modificalo:

```json
{
  "name": "personal-notes-api",
  "version": "0.1.0",
  "description": "Personal notes API",
  "type": "module",
  "scripts": {
    "dev": "tsx watch src/server.ts",
    "build": "tsc",
    "start": "node dist/server.js",
    "test": "vitest run --coverage",
    "test:watch": "vitest",
    "lint": "eslint . --ext .ts",
    "lint:fix": "eslint . --ext .ts --fix",
    "format": "prettier --write .",
    "format:check": "prettier --check .",
    "typecheck": "tsc --noEmit",
    "migrate": "tsx src/db/migrate.ts"
  },
  "license": "MIT"
}
```

### Installa dipendenze

Runtime:

```bash
npm install express pg
npm install --save-dev typescript @types/node @types/express @types/pg tsx vitest @vitest/coverage-v8 supertest @types/supertest eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin prettier
```

### Configurazione TypeScript

Crea `tsconfig.json`:

```bash
cat > tsconfig.json << 'EOF'
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "resolveJsonModule": true,
    "forceConsistentCasingInFileNames": true,
    "isolatedModules": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "declaration": true,
    "sourceMap": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "**/*.test.ts"]
}
EOF
```

### Configurazione ESLint

Crea `.eslintrc.json`:

```json
{
  "parser": "@typescript-eslint/parser",
  "plugins": ["@typescript-eslint"],
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended"
  ],
  "env": {
    "node": true,
    "es2022": true
  },
  "rules": {
    "@typescript-eslint/no-unused-vars": ["error", { "argsIgnorePattern": "^_" }]
  },
  "ignorePatterns": ["dist", "node_modules", "coverage"]
}
```

### Configurazione Prettier

Crea `.prettierrc.json`:

```json
{
  "semi": true,
  "singleQuote": true,
  "trailingComma": "es5",
  "printWidth": 100,
  "tabWidth": 2,
  "endOfLine": "lf"
}
```

### Struttura cartelle

```bash
mkdir -p src/{routes,services,db,middleware,utils} tests
echo "console.log('Personal Notes API starting...');" > src/server.ts
```

### Primo test

Crea `tests/sanity.test.ts`:

```typescript
import { describe, it, expect } from 'vitest';

describe('sanity', () => {
  it('should run tests', () => {
    expect(1 + 1).toBe(2);
  });
});
```

Esegui:

```bash
npm test
```

Deve passare. Se sì, lo scaffolding è in piedi.

### Primo commit

```bash
git add .
git status              # verifica cosa stai per committare
git commit -m "chore: initial project scaffolding"
```

Aggiungi il remote e push:

```bash
git remote add origin git@github.com:<user>/personal-notes-api.git
git push -u origin main
```

Ricarica la pagina del repo su GitHub. Tutti i file sono lì.

## 3.4 — Setup Docker

### `Dockerfile`

```bash
cat > Dockerfile << 'EOF'
# syntax=docker/dockerfile:1.7

# ---------- Stage 1: dependencies ----------
FROM node:20-alpine AS deps
WORKDIR /app
COPY package.json package-lock.json ./
RUN --mount=type=cache,target=/root/.npm npm ci

# ---------- Stage 2: build ----------
FROM node:20-alpine AS build
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build && npm prune --production

# ---------- Stage 3: runtime ----------
FROM node:20-alpine AS runtime
WORKDIR /app
ENV NODE_ENV=production

# Crea utente non-root
RUN addgroup -S app && adduser -S app -G app

COPY --from=build --chown=app:app /app/node_modules ./node_modules
COPY --from=build --chown=app:app /app/dist ./dist
COPY --from=build --chown=app:app /app/package.json ./

USER app
EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --quiet --tries=1 --spider http://localhost:3000/health || exit 1

CMD ["node", "dist/server.js"]
EOF
```

### `.dockerignore`

```bash
cat > .dockerignore << 'EOF'
.git
.github
node_modules
dist
build
coverage
.env
.env.*
*.log
.DS_Store
.idea
.vscode
docs
*.md
!README.md
Dockerfile
docker-compose*.yml
.eslintrc*
.prettierrc*
tests
EOF
```

### `docker-compose.yml` per dev

```bash
cat > docker-compose.yml << 'EOF'
services:
  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: dev
      POSTGRES_PASSWORD: dev
      POSTGRES_DB: notes_dev
    volumes:
      - db_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U dev"]
      interval: 5s
      timeout: 5s
      retries: 5
    ports:
      - "5432:5432"

volumes:
  db_data:
EOF
```

Lancia il database:

```bash
docker compose up -d
docker compose ps           # verifica che il container db sia "healthy"
docker compose logs db      # vedi log se serve
```

Stop:

```bash
docker compose down
```

(Riavvialo dopo, lo useremo.)

## 3.5 — Secret management

### File `.env.example`

Documenta quali variabili servono, **senza valori reali**:

```bash
cat > .env.example << 'EOF'
# Application
NODE_ENV=development
PORT=3000
LOG_LEVEL=info

# Database
DATABASE_URL=postgres://dev:dev@localhost:5432/notes_dev

# Auth
API_TOKEN=

# Optional
SENTRY_DSN=
EOF
```

### File `.env` (locale, **non committare**)

```bash
cp .env.example .env
```

Apri `.env` e riempi i valori veri:

```
NODE_ENV=development
PORT=3000
LOG_LEVEL=debug
DATABASE_URL=postgres://dev:dev@localhost:5432/notes_dev
API_TOKEN=dev-token-please-change-me
SENTRY_DSN=
```

Verifica che `.env` sia ignorato:

```bash
git status              # .env NON deve apparire
git check-ignore .env   # deve restituire ".env"
```

Se `.env` apparisse in git status, **fermati subito**: il `.gitignore` non sta funzionando.

### Secret manager (per il futuro)

In dev, `.env` locale basta. Per staging/production, dovrai usare un secret manager o le variabili d'ambiente del provider di hosting. Ne riparleremo in Parte 5.

## 3.6 — Pre-commit hooks

I pre-commit hook eseguono controlli prima di permettere un commit. Vogliamo: lint, format, type check, secret scan.

### Configurazione lefthook

Crea `lefthook.yml`:

```bash
cat > lefthook.yml << 'EOF'
pre-commit:
  parallel: true
  commands:
    lint:
      glob: "*.{ts,js}"
      run: npx eslint {staged_files}
    format:
      glob: "*.{ts,js,json,md,yml,yaml}"
      run: npx prettier --check {staged_files}
    typecheck:
      glob: "*.ts"
      run: npx tsc --noEmit
    secrets:
      run: gitleaks protect --staged --redact -v

commit-msg:
  commands:
    commitlint:
      run: |
        if ! head -1 {1} | grep -qE '^(feat|fix|docs|style|refactor|perf|test|chore|ci|build)(\(.+\))?!?: .+'; then
          echo "Commit message must follow Conventional Commits format."
          echo "Example: feat(auth): add login endpoint"
          exit 1
        fi
EOF
```

Installa gli hook nel repo:

```bash
lefthook install
```

Test: prova a committare un file mal formattato:

```bash
echo "const   x= 1" >> src/server.ts
git add src/server.ts
git commit -m "wip"   # FALLISCE perché format check fallisce e commit message non conventional
```

Risolvi:

```bash
git restore --staged src/server.ts
git restore src/server.ts
```

### Skip in emergenza

`git commit --no-verify` salta gli hook. Usalo **solo** per WIP su branch di lavoro, mai per commit destinati a main.

## 3.7 — Branch protection rules su GitHub

Vai su GitHub → repo → Settings → Branches → Add rule:

- Branch name pattern: `main`.
- Spunta:
  - **Require a pull request before merging**.
  - **Require status checks to pass before merging** → li configureremo dopo (CI).
  - **Require branches to be up to date before merging**.
  - **Require linear history**.
  - **Do not allow bypassing the above settings**.
- Salva.

Da ora, anche tu non puoi pushare direttamente su main: devi passare da una PR.

## 3.8 — CI/CD con GitHub Actions

Crea `.github/workflows/ci.yml`:

```bash
mkdir -p .github/workflows
cat > .github/workflows/ci.yml << 'EOF'
name: CI

on:
  pull_request:
  push:
    branches: [main]

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16-alpine
        env:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: notes_test
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version-file: .nvmrc
          cache: npm

      - run: npm ci

      - name: Lint
        run: npm run lint

      - name: Format check
        run: npm run format:check

      - name: Typecheck
        run: npm run typecheck

      - name: Test
        run: npm test
        env:
          DATABASE_URL: postgres://test:test@localhost:5432/notes_test
          NODE_ENV: test

  build:
    runs-on: ubuntu-latest
    needs: test
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version-file: .nvmrc
          cache: npm
      - run: npm ci
      - run: npm run build

  docker:
    runs-on: ubuntu-latest
    needs: build
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v4
      - uses: docker/setup-buildx-action@v3
      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - uses: docker/metadata-action@v5
        id: meta
        with:
          images: ghcr.io/${{ github.repository }}
          tags: |
            type=ref,event=branch
            type=sha
            type=semver,pattern={{version}}
      - uses: docker/build-push-action@v5
        with:
          context: .
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ steps.meta.outputs.tags }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run gitleaks
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
EOF
```

Cosa fa questo workflow:

- **Trigger**: parte su ogni PR e push su main.
- **Job test**: avvia un container Postgres temporaneo, installa dipendenze, esegue lint, format check, typecheck, test.
- **Job build**: compila TypeScript.
- **Job docker**: costruisce l'immagine Docker, la taggа, e (solo se non è PR) la pusha su GHCR.
- **Job security**: esegue gitleaks per cercare secret.

## 3.9 — Dependabot

Crea `.github/dependabot.yml`:

```bash
cat > .github/dependabot.yml << 'EOF'
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
    open-pull-requests-limit: 5
    groups:
      patch-and-minor:
        update-types: ["patch", "minor"]
    labels:
      - "dependencies"

  - package-ecosystem: "docker"
    directory: "/"
    schedule:
      interval: "weekly"

  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
EOF
```

## 3.10 — Templates GitHub

### PR template

```bash
cat > .github/pull_request_template.md << 'EOF'
## Cosa cambia
<!-- Descrivi in breve cosa fa questa PR -->

## Perché
<!-- Motivazione, link a issue -->

Closes #

## Come testare
<!-- Passi per verificare in locale -->

## Checklist
- [ ] Test aggiunti/aggiornati
- [ ] Coverage rispettata
- [ ] Documentazione aggiornata
- [ ] CHANGELOG.md aggiornato
- [ ] /verification-loop PASS in locale
- [ ] Security review se applicabile
EOF
```

### Issue templates

```bash
mkdir -p .github/ISSUE_TEMPLATE
cat > .github/ISSUE_TEMPLATE/bug_report.md << 'EOF'
---
name: Bug report
about: Segnala un bug
labels: bug
---

## Descrizione
<!-- Cosa succede di sbagliato -->

## Passi per riprodurre
1.
2.
3.

## Comportamento atteso

## Comportamento attuale

## Ambiente
- Versione:
- OS:
EOF

cat > .github/ISSUE_TEMPLATE/feature_request.md << 'EOF'
---
name: Feature request
about: Proponi una nuova feature
labels: feat
---

## Cosa
<!-- Descrizione della feature -->

## Perché
<!-- Caso d'uso, problema risolto -->

## Criteri di accettazione
- [ ]
- [ ]
EOF
```

### CODEOWNERS

```bash
cat > .github/CODEOWNERS << 'EOF'
* @<tuo-username>
EOF
```

## 3.11 — Issue tracking / Kanban

GitHub Projects è il default semplice:

1. Vai su repo → Projects → New project.
2. Template "Board".
3. Colonne: `Backlog`, `Todo`, `In progress`, `Review`, `Done`.

Da ora, ogni feature parte da una issue collegata al board.

## 3.12 — Cartella `docs/` completa

Crea i file placeholder:

```bash
mkdir -p docs/{adr,incidents,spikes}
touch docs/deploy.md docs/rollback.md docs/secrets.md docs/observability.md
touch docs/tech-debt.md docs/security-exceptions.md docs/runbook.md docs/conventions.md
```

Riempili minimamente:

```bash
cat > docs/runbook.md << 'EOF'
# Runbook

## Cosa fare quando qualcosa va male

### Health check rosso
1. Controlla i log: `ssh prod-server 'docker compose logs app --tail 100'`
2. Se DB-related: vedi sezione DB.
3. Se non chiaro entro 5 minuti: rollback (vedi rollback.md).

### Disk full
1. ...

### Spesa cloud baseline
- Mese tipo: €X
- Allarme se: > €X * 1.5
EOF
```

(Aggiungerai contenuto man mano.)

## 3.13 — Verifica finale del setup

Esegui:

```bash
# Tutto il toolchain
npm run lint
npm run format:check
npm run typecheck
npm test

# Docker
docker compose up -d
docker compose ps
docker compose down

# Git status pulito
git status
```

Se tutti i comandi sopra restituiscono OK, il setup è completo.

Commit finale:

```bash
git add .
git commit -m "chore: complete project setup with Docker, CI, and tooling"
```

Apri una PR per portare tutto su main:

```bash
git checkout -b chore/initial-setup
git push -u origin chore/initial-setup
```

Vai su GitHub → apri PR → aspetta CI verde → squash and merge.

CI fallirà perché abbiamo `main` come default ma stiamo aprendo PR su `main` da `main`. Fix:

```bash
# Sul main pre-commit: torna allo stato pulito
git checkout main
git reset --hard origin/main   # se hai pushato già

# Lavora su un branch
git checkout -b chore/initial-setup
git push -u origin chore/initial-setup
```

Apri PR `chore/initial-setup` → `main`. CI verde → squash merge.

A questo punto **tutto il setup è in produzione**. Hai un repository professionale completo. Hai investito 3-4 ore. Le ripagherà in centinaia di ore future.

---

# Parte 4 — Il Tuo Primo Progetto: Sviluppo della Prima Feature

Ora svilupperemo la prima feature. Useremo il workflow Fase 0 → 10 dal documento di riferimento, ma con **spiegazioni complete** per il principiante.

Feature scelta: **CRUD note con autenticazione Bearer token**.

## 4.1 — Fase 0: Apri una issue

Vai su GitHub → repo → Issues → New issue → "Feature request":

- **Titolo**: `feat: CRUD endpoints for notes with Bearer auth`
- **Descrizione**:
  ```
  ## Cosa
  Endpoint REST per creare, leggere, aggiornare, cancellare note. Autenticazione tramite Bearer token (singolo token per single-user).

  ## Perché
  Core MVP del progetto. Senza questo, non c'è API.

  ## Criteri di accettazione
  - POST /api/notes — crea nota.
  - GET /api/notes — lista note.
  - GET /api/notes/:id — singola nota.
  - PATCH /api/notes/:id — aggiorna nota.
  - DELETE /api/notes/:id — cancella nota.
  - Tutti gli endpoint richiedono header `Authorization: Bearer <token>`.
  - 401 senza token, 403 se token sbagliato.
  - Response in JSON, errori in formato standard.
  - Test coverage >= 70% sul codice di business, 100% sul middleware di auth.
  - OpenAPI doc generata.
  ```
- Labels: `feat`.
- Salva.

Numero issue assegnato (es. #1).

## 4.2 — Fase 1: Search-first

Apri Claude Code nel terminale dentro la cartella del progetto:

```bash
cd ~/dev/personal-notes-api
claude
```

Chiedi:

```
Devo costruire un'API REST CRUD per gestire note con autenticazione Bearer token (singolo token statico per single-user).
Stack: Node 20, TypeScript, Express, PostgreSQL, vitest per i test.
Features:
- POST/GET/GET-by-id/PATCH/DELETE /api/notes
- Middleware Bearer auth
- Validazione input
- Logger strutturato

Usa search-first per trovare librerie, boilerplate o approcci esistenti prima di scrivere qualsiasi cosa.
```

Claude userà i suoi tool per cercare e ti proporrà:

- Librerie per validazione (Zod, Joi, AJV).
- Logger (pino, winston).
- Possibili boilerplate completi.

Confronta. Per la nostra guida, usiamo: **Zod** per validazione, **pino** per logging, **dotenv** per le env, **node-postgres (pg)** per il DB.

Verifica licenze (sono tutte MIT/ISC, ok).

```bash
npm install zod pino dotenv
npm install --save-dev pino-pretty
```

## 4.3 — Fase 2: Council (skip per questa feature)

`/council` è per decisioni difficili. Qui le scelte sono ovvie. Salta.

## 4.4 — Fase 3: Pianificazione

In Claude Code:

```
Pianifica l'implementazione della issue #1.
Stack: Node 20 + Express + TypeScript + Postgres.
Criteri di accettazione: vedi issue.
Produci: architettura, struttura file, task list ordinata, dipendenze, rischi noti.
```

Salva il piano in `docs/plan-feature-notes.md` (o aggiornalo nella issue stessa).

Poi:

```
/compact Focus on implementing notes CRUD with Bearer auth on Express + Postgres.
```

## 4.5 — Branch

```bash
git checkout main
git pull
git checkout -b feat/notes-crud-auth-1
```

Convenzione: `<type>/<short-desc>-<issue-number>`.

## 4.6 — Fase 4: Sviluppo (TDD)

Useremo TDD. Per la nostra feature:

- **Auth middleware**: 100% coverage (security-critical).
- **Routes/services**: 80% coverage.
- **DB layer**: 70% coverage.

In Claude Code:

```
Implementa il middleware di Bearer auth seguendo tdd-workflow:
- Prima i test (unit)
- Poi l'implementazione
- Coverage 100% inclusi i casi di errore (no header, header malformato, token sbagliato)

Usa le seguenti convenzioni:
- File in src/middleware/auth.ts
- Test in tests/middleware/auth.test.ts
- Express middleware standard
- Token letto da env API_TOKEN
- Response 401 se manca header, 403 se token errato, formato {error: string}
```

Claude scriverà:

1. `tests/middleware/auth.test.ts` con casi di test.
2. `src/middleware/auth.ts` con l'implementazione minima per farli passare.

Esempio di come potrebbe risultare il test:

```typescript
import { describe, it, expect, vi } from 'vitest';
import { authMiddleware } from '../../src/middleware/auth.js';
import type { Request, Response, NextFunction } from 'express';

const makeReqRes = (auth?: string) => {
  const req = { headers: { authorization: auth } } as Request;
  const res = {
    status: vi.fn().mockReturnThis(),
    json: vi.fn().mockReturnThis(),
  } as unknown as Response;
  const next = vi.fn() as NextFunction;
  return { req, res, next };
};

describe('authMiddleware', () => {
  beforeEach(() => {
    process.env.API_TOKEN = 'secret';
  });

  it('returns 401 if Authorization header is missing', () => {
    const { req, res, next } = makeReqRes();
    authMiddleware(req, res, next);
    expect(res.status).toHaveBeenCalledWith(401);
    expect(next).not.toHaveBeenCalled();
  });

  it('returns 401 if header is malformed', () => {
    const { req, res, next } = makeReqRes('NotBearer abc');
    authMiddleware(req, res, next);
    expect(res.status).toHaveBeenCalledWith(401);
  });

  it('returns 403 if token is wrong', () => {
    const { req, res, next } = makeReqRes('Bearer wrong');
    authMiddleware(req, res, next);
    expect(res.status).toHaveBeenCalledWith(403);
  });

  it('calls next() if token is valid', () => {
    const { req, res, next } = makeReqRes('Bearer secret');
    authMiddleware(req, res, next);
    expect(next).toHaveBeenCalled();
    expect(res.status).not.toHaveBeenCalled();
  });
});
```

E l'implementazione:

```typescript
import type { Request, Response, NextFunction } from 'express';

export function authMiddleware(req: Request, res: Response, next: NextFunction): void {
  const header = req.headers.authorization;
  if (!header || !header.startsWith('Bearer ')) {
    res.status(401).json({ error: 'Authorization header missing or malformed' });
    return;
  }
  const token = header.slice('Bearer '.length);
  if (token !== process.env.API_TOKEN) {
    res.status(403).json({ error: 'Invalid token' });
    return;
  }
  next();
}
```

Lancia i test:

```bash
npm test
```

Devono passare. Verifica coverage:

```
/test-coverage
```

Salva un checkpoint:

```
/checkpoint
```

Commit:

```bash
git add src/middleware/auth.ts tests/middleware/auth.test.ts
git commit -m "feat(auth): add Bearer token middleware"
```

### Continua con le altre componenti

Ripeti il pattern per:

1. **Migrazione DB** (`src/db/migrate.ts`): crea tabella `notes`.
2. **Repository** (`src/db/notes.repo.ts`): funzioni `createNote`, `getNotes`, `getNoteById`, `updateNote`, `deleteNote`.
3. **Service** (`src/services/notes.service.ts`): logica di business.
4. **Routes** (`src/routes/notes.ts`): endpoint Express.
5. **Server** (`src/server.ts`): assemblaggio Express con middleware e routes.
6. **Validation** con Zod.

Per ognuno:

```
Implementa <componente> seguendo tdd-workflow.
Coverage target: <X%>.
Vincoli: <specificare>
```

Checkpoint dopo ogni step verde.

## 4.7 — Database migrations

Per gestire schema DB in modo versionato. Usiamo un approccio semplice (pochi file SQL numerati).

Crea `src/db/migrations/001_create_notes.sql`:

```sql
CREATE TABLE notes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  title TEXT NOT NULL,
  body TEXT NOT NULL DEFAULT '',
  tags TEXT[] NOT NULL DEFAULT '{}',
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_notes_created_at ON notes(created_at DESC);
```

E `src/db/migrate.ts` che la applica. Per progetti più seri si usa Knex, Prisma Migrate, ecc. Per questo MVP, uno script semplice basta.

**Regola d'oro**: ogni migration ha una **down** o un piano di rollback. Mai migration distruttive senza piano.

## 4.8 — Fase 5: Code review

Quando tutta la feature è implementata e i test passano:

```
/code-review
```

Leggi i suggerimenti. Se ce ne sono di validi, applicali. Se il codice ti sembra "tutto ok", forza:

```
Trova i 3 punti più deboli di questa implementazione. Non essere indulgente.
```

Per refactor mirato:

```
/refactor-clean
```

(Timebox 1 ora, ricorda.)

## 4.9 — Fase 6: Security review

Questa feature ha auth e DB queries. Security review obbligatoria.

```
/security-review
```

Aggiungi contesto se serve:

```
Aggiungi a quanto già controllato anche: il middleware di auth confronta token con stringa costante, controlla se è vulnerabile a timing attack. L'endpoint POST accetta JSON arbitrario, controlla la validazione Zod.
```

Scansione automatica:

```bash
npx ecc-agentshield scan
```

Se trovi HIGH/CRITICAL: stop, fix, ripeti.

Container scan:

```bash
docker build -t personal-notes-api:dev .
trivy image --severity HIGH,CRITICAL personal-notes-api:dev
```

## 4.10 — Fase 7: Verification

```
/verification-loop
```

Esegue: build, typecheck, lint, test con coverage, security scan. PASS richiesto.

## 4.11 — Fase 8: Commit, PR, merge

### Pulizia commit

Sei sul branch `feat/notes-crud-auth-1`. Probabilmente hai vari commit "feat(...)", "test(...)", "chore(...)". Vanno bene. Aggiorna col main:

```bash
git fetch origin main
git rebase origin/main
```

### Push

```bash
git push -u origin feat/notes-crud-auth-1
```

### PR

```bash
gh pr create --fill
```

Oppure su GitHub: New pull request → seleziona branch → usa il template PR (è già configurato).

```
/prp-pr
```

Genera descrizione completa.

### Self-review

```
/review-pr
```

### Aspetta CI verde

Il workflow CI parte automaticamente. Aspetta che tutti i job siano verdi.

### Merge

GitHub → tasto "Squash and merge" → conferma. Branch eliminato automaticamente.

### Tag (se è una milestone)

Se la PR completa un MVP o una versione:

```bash
git checkout main
git pull
git tag -a v0.1.0 -m "Release v0.1.0: notes CRUD with Bearer auth"
git push origin v0.1.0
```

Su GitHub → Releases → Draft new release → seleziona il tag → "Generate release notes".

## 4.12 — Fase 9: Documentazione

```
/update-docs
```

Verifica e completa a mano:

- README aggiornato se i comandi sono cambiati.
- CHANGELOG.md sotto `[Unreleased]`:
  ```
  ### Added
  - Notes CRUD endpoints (POST, GET, GET by ID, PATCH, DELETE).
  - Bearer token authentication middleware.
  - Database migrations system.
  ```
- API docs: genera OpenAPI spec da codice (esiste libreria `swagger-jsdoc`) o scrivi `docs/api.md` a mano.
- ADR se hai preso decisioni architetturali.

Commit nello stesso PR (se ancora aperta) o in una PR docs separata.

## 4.13 — Fase 10: Fine sessione

```
/learn
/save-session
/evolve
```

```bash
git push   # se hai commit non pushati
```

A fine giornata, anche se la feature non è finita, **push**. Single point of failure mitigation.

A inizio sessione successiva:

```
/resume-session
```

---

# Parte 5 — Deploy e Operazioni

Mostro un percorso completo per portare il progetto in produzione. Hosting di esempio: una VPS (DigitalOcean Droplet, Hetzner Cloud, ecc.) con Docker. Adatta al tuo provider.

## 5.1 — Strategia di deploy

Per progetto piccolo: **recreate**. Stop container vecchio, start nuovo. Downtime di pochi secondi, accettabile per MVP single-user.

Più avanti, quando avrai utenti reali, valuta rolling deploy con almeno 2 istanze + load balancer.

## 5.2 — Setup staging

### Provider VPS

Esempio Hetzner Cloud:

1. Crea account.
2. Crea un progetto "personal-notes-api".
3. Crea un Server → CX22 (€4-5/mese) → Ubuntu 24.04 → Frankfurt → SSH key (la tua).
4. Annota IP pubblico.

Ripeti per `staging` e `production`.

### Setup iniziale del server (entrambi gli ambienti)

```bash
ssh root@<IP>
```

Una volta loggato:

```bash
# Aggiorna sistema
apt update && apt upgrade -y

# Crea utente non-root
adduser deploy
usermod -aG sudo deploy

# Copia la SSH key
mkdir -p /home/deploy/.ssh
cp ~/.ssh/authorized_keys /home/deploy/.ssh/
chown -R deploy:deploy /home/deploy/.ssh
chmod 700 /home/deploy/.ssh
chmod 600 /home/deploy/.ssh/authorized_keys

# Disabilita login root via SSH
nano /etc/ssh/sshd_config
# Cambia: PermitRootLogin no
# Cambia: PasswordAuthentication no
systemctl restart sshd

# Firewall (UFW)
ufw allow OpenSSH
ufw allow 80/tcp
ufw allow 443/tcp
ufw enable

# Installa Docker
curl -fsSL https://get.docker.com | sh
usermod -aG docker deploy

# Logout, riconnetti come deploy
exit
ssh deploy@<IP>
docker --version
```

### Deploy della prima versione

Sul server, crea cartella e file:

```bash
mkdir -p /home/deploy/app
cd /home/deploy/app
```

Crea `docker-compose.prod.yml`:

```yaml
services:
  app:
    image: ghcr.io/<user>/personal-notes-api:v0.1.0
    restart: unless-stopped
    env_file: .env
    ports:
      - "3000:3000"
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:16-alpine
    restart: unless-stopped
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASS}
      POSTGRES_DB: ${DB_NAME}
    volumes:
      - db_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER}"]
      interval: 5s
      timeout: 5s
      retries: 5

volumes:
  db_data:
```

Crea `.env` (sul server, **non in repo**):

```
NODE_ENV=production
PORT=3000
LOG_LEVEL=info
DATABASE_URL=postgres://notes:<password-forte>@db:5432/notes
DB_USER=notes
DB_PASS=<password-forte>
DB_NAME=notes
API_TOKEN=<token-forte>
```

Genera password e token:

```bash
openssl rand -hex 32        # per password DB e API token
```

Login al GHCR per scaricare l'immagine:

```bash
echo $GITHUB_TOKEN | docker login ghcr.io -u <user> --password-stdin
```

(Crea un GitHub Personal Access Token con scope `read:packages` su GitHub → Settings → Developer settings.)

Lancia:

```bash
docker compose -f docker-compose.prod.yml pull
docker compose -f docker-compose.prod.yml up -d
docker compose -f docker-compose.prod.yml logs -f
```

Test: dal tuo PC:

```bash
curl http://<IP>:3000/health
```

Deve restituire 200.

## 5.3 — SSL/TLS, dominio, DNS

### Dominio

Compra un dominio su Cloudflare Registrar, Porkbun, o simili. Esempio: `notes.example.com`.

### DNS

Punta `notes.example.com` (record A) all'IP del server. Su Cloudflare: DNS → Add record → A → name `notes` → IPv4 server → proxy off (per prima richiesta certificato).

### Certificato HTTPS con Caddy

Caddy è un reverse proxy che gestisce certificati Let's Encrypt automaticamente.

Aggiungi al `docker-compose.prod.yml`:

```yaml
services:
  caddy:
    image: caddy:2-alpine
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile:ro
      - caddy_data:/data
      - caddy_config:/config

volumes:
  caddy_data:
  caddy_config:
```

E rimuovi `ports: "3000:3000"` da `app` (il traffico passerà per Caddy).

Crea `Caddyfile`:

```
notes.example.com {
    reverse_proxy app:3000
    encode gzip
    log {
        output file /data/access.log {
            roll_size 10mb
            roll_keep 10
        }
    }
}
```

Riavvia:

```bash
docker compose -f docker-compose.prod.yml up -d
```

Caddy richiede il certificato Let's Encrypt automaticamente. Verifica:

```bash
curl https://notes.example.com/health
```

## 5.4 — Observability

### Logs strutturati

Già configurato con `pino` (vedi Parte 4). Verifica che siano JSON in produzione:

```bash
docker compose -f docker-compose.prod.yml logs app --tail 20
```

### Metrics

Per progetto piccolo, integra **Better Stack** (free tier) o **UptimeRobot** (gratuito) per monitoring uptime + alert.

Vai su https://betterstack.com/ → crea monitor → URL `https://notes.example.com/health` → check ogni 1 minuto → alert email se down per > 2 minuti.

### Error tracking con Sentry

1. Crea progetto su https://sentry.io/ (free tier).
2. Ottieni DSN.
3. Aggiungi a `.env` produzione: `SENTRY_DSN=...`.
4. Integra in codice:
   ```typescript
   import * as Sentry from '@sentry/node';
   Sentry.init({ dsn: process.env.SENTRY_DSN, tracesSampleRate: 0.1 });
   ```
5. Configura source maps upload in build per stacktrace leggibili.

## 5.5 — Backup

### Backup automatico DB

Sul server, crea `/home/deploy/scripts/backup.sh`:

```bash
#!/bin/bash
set -e
DATE=$(date +%Y%m%d-%H%M%S)
BACKUP_DIR=/home/deploy/backups
mkdir -p $BACKUP_DIR

docker compose -f /home/deploy/app/docker-compose.prod.yml exec -T db \
  pg_dump -U notes notes | gzip > $BACKUP_DIR/notes-$DATE.sql.gz

# Mantieni solo ultimi 30 giorni
find $BACKUP_DIR -name "notes-*.sql.gz" -mtime +30 -delete

# Upload off-site (es. Backblaze B2 con rclone, o AWS S3)
# rclone copy $BACKUP_DIR/notes-$DATE.sql.gz b2:my-backups/notes/
```

Rendilo eseguibile e schedulalo:

```bash
chmod +x /home/deploy/scripts/backup.sh
crontab -e
# Aggiungi:
# 0 3 * * * /home/deploy/scripts/backup.sh >> /home/deploy/backup.log 2>&1
```

### Test di restore mensile

Crea `/home/deploy/scripts/restore-test.sh` e schedulalo mensilmente. Restore in un container temporaneo, verifica che il dump sia leggibile.

## 5.6 — Procedure documentate

Riempi `docs/deploy.md`:

```markdown
# Deploy procedure

## Production deploy v.X.Y.Z

### Pre-deploy checklist
- [ ] Backup DB: `ssh prod 'sudo -u deploy /home/deploy/scripts/backup.sh'`
- [ ] Tag rilasciato su GitHub
- [ ] CI verde su tag
- [ ] Smoke test su staging

### Deploy
```bash
ssh deploy@prod-server
cd /home/deploy/app
# Aggiorna immagine in docker-compose.prod.yml: vX.Y.Z
nano docker-compose.prod.yml
docker compose -f docker-compose.prod.yml pull
docker compose -f docker-compose.prod.yml up -d
docker compose -f docker-compose.prod.yml logs -f app
```

### Verifica
```bash
curl https://notes.example.com/health
```

### Annotazione
Annotare versione precedente (per rollback) in docs/runbook.md.
```

E `docs/rollback.md`:

```markdown
# Rollback procedure

## Tempo stimato: 2 minuti

```bash
ssh deploy@prod-server
cd /home/deploy/app
# Modifica docker-compose.prod.yml: ripristina versione precedente
nano docker-compose.prod.yml
docker compose -f docker-compose.prod.yml pull
docker compose -f docker-compose.prod.yml up -d
curl https://notes.example.com/health
```

Se rollback non basta (es. migration distruttiva): restore DB da backup.
```

## 5.7 — Cost monitoring

Su Hetzner / DigitalOcean / cloud preferito:

- Configura budget alert al 80%, 100%, 150%.
- Annota in `docs/runbook.md` baseline mensile (es. €15/mese: VPS €5 + dominio €1 + altro).
- Review primo del mese: 15 minuti.

---

# Parte 6 — Manutenzione Continua

## 6.1 — Settimanale (lunedì mattina, 30 min)

- Review e merge PR Dependabot:
  ```bash
  gh pr list --label dependencies
  ```
  Patch + minor: merge se CI verde. Major: leggi changelog, ADR se necessario.
- Triage issue aperte: chiudi obsolete, etichetta nuove.
- Check log/error tracking della settimana.

## 6.2 — Mensile (primo del mese, 2 ore)

- Cost review.
- Backup restore test.
- `npm audit`, GitHub Security tab review.
- `/instinct-export` → backup in repo separato.
- Review `docs/tech-debt.md`.
- Aggiornamento documentazione obsoleta.

## 6.3 — Trimestrale (mezza giornata)

- Performance test con k6:
  ```bash
  k6 run --vus 50 --duration 5m scripts/load-test.js
  ```
- Disaster recovery drill: restore DB su staging, verifica integrità.
- Major dependency review.
- ADR review.
- Cleanup branch stale, tag inutili, immagini Docker vecchie.
- Renewal check: dominio, certificati (auto), abbonamenti.

## 6.4 — Annuale

- Rotazione di tutti i secret long-lived.
- Review licenza progetto.
- Penetration test (anche solo OWASP ZAP automatico).
- Aggiornamento runtime (Node 20 → 22).
- Revisione del workflow stesso: cosa hai cambiato in pratica?

---

# Parte 7 — Riferimenti

## 7.1 — Glossario completo

**ADR (Architecture Decision Record)**: documento breve che cattura una decisione architetturale, opzioni considerate, motivazione, conseguenze.

**API (Application Programming Interface)**: interfaccia che un software espone per essere usato da altri software. Comunemente HTTP/REST in ambito web.

**Branch (Git)**: linea di sviluppo separata.

**Build**: processo di trasformazione del codice sorgente in artefatti eseguibili (compilato, bundle, immagine Docker).

**CA (Certificate Authority)**: ente che firma certificati TLS. Let's Encrypt è una CA gratuita.

**CI/CD (Continuous Integration/Continuous Delivery)**: pratiche e automazioni per integrare e deployare codice frequentemente.

**Commit (Git)**: snapshot immutabile delle modifiche.

**Container**: istanza in esecuzione di un'immagine Docker.

**Coverage**: percentuale di codice eseguita dai test.

**CVE (Common Vulnerabilities and Exposures)**: ID univoco di una vulnerabilità nota.

**Dependabot**: tool GitHub che apre PR per aggiornare dipendenze.

**Docker**: piattaforma di containerizzazione.

**Dockerfile**: file di istruzioni per costruire un'immagine Docker.

**End-to-end test (e2e)**: test che verifica il flusso utente completo.

**Environment variable**: variabile di configurazione letta dal sistema operativo, esposta al programma.

**Hash (Git)**: identificatore univoco di un commit, calcolato con SHA-1.

**Health check**: endpoint che dichiara se il servizio è funzionante.

**HTTPS**: HTTP cifrato con TLS.

**Image (Docker)**: snapshot immutabile di un filesystem + config per eseguire un'app.

**Issue (GitHub)**: ticket per tracciare un task, bug, feature.

**Kanban**: metodologia di gestione visuale del lavoro (board con colonne).

**Lefthook**: tool per gestire pre-commit hook.

**Lint**: analisi statica del codice per trovare errori e anti-pattern.

**Lock file**: file che fissa esattamente le versioni delle dipendenze.

**Merge (Git)**: unione di due branch.

**Migration (DB)**: file che descrive una modifica allo schema del database.

**MVP (Minimum Viable Product)**: prima versione funzionante, riducente al minimo le feature.

**npm**: package manager di Node.js.

**Observability**: capacità di capire cosa sta facendo un sistema in produzione.

**OWASP**: organizzazione che cura best practice e top vulnerability list per sicurezza web.

**Package manager**: tool che gestisce dipendenze del progetto.

**PostgreSQL**: database relazionale.

**Pull Request (PR)**: richiesta di merge di un branch in un altro su GitHub.

**Rebase (Git)**: operazione che riapplica commit di un branch su un altro.

**Registry**: server che ospita immagini Docker (es. Docker Hub, GHCR).

**Remote (Git)**: copia del repository su un server.

**Repository (repo)**: cartella di un progetto sotto controllo Git.

**Revert (Git)**: crea un nuovo commit che annulla un commit precedente.

**Rollback**: tornare alla versione precedente di un'app deployata.

**Runtime**: ambiente di esecuzione del codice (Node, Python, JVM, ecc.).

**Secret**: valore sensibile (password, API key) che dà accesso a qualcosa.

**SemVer (Semantic Versioning)**: schema MAJOR.MINOR.PATCH.

**Shell**: programma che interpreta comandi (Bash, Zsh, PowerShell).

**SSH (Secure Shell)**: protocollo per connessioni remote sicure.

**Staging**: ambiente identico a production, usato per testing pre-deploy.

**TDD (Test-Driven Development)**: tecnica di scrivere test prima dell'implementazione.

**TLS (Transport Layer Security)**: protocollo per cifratura traffico (HTTPS).

**Trivy**: scanner di vulnerabilità per immagini Docker.

**TypeScript**: superset di JavaScript con tipizzazione statica.

**Unit test**: test di una singola funzione/classe in isolamento.

**Volume (Docker)**: meccanismo per persistere dati fuori dal container.

**WSL2 (Windows Subsystem for Linux 2)**: Linux dentro Windows.

**Zod**: libreria TypeScript per validazione di schema.

## 7.2 — Comandi cheat-sheet

### Git essenziali

```bash
git status                        # stato corrente
git add <file>                    # stage
git add .                         # stage tutto
git commit -m "feat: ..."         # commit
git push                          # push al remote
git pull                          # pull dal remote
git checkout -b feat/x            # crea branch
git switch main                   # cambia branch
git log --oneline                 # storia compatta
git diff                          # differenze non staged
git diff --staged                 # differenze staged
git restore <file>                # scarta modifiche
git reset --hard origin/main      # reset al main remoto (DISTRUTTIVO)
git stash                         # metti da parte modifiche
git stash pop                     # riprendi modifiche stashate
git rebase main                   # rebase sul main
git merge --abort                 # annulla un merge in corso
```

### Docker essenziali

```bash
docker build -t name .            # costruisci immagine
docker run -p 3000:3000 name      # esegui container
docker ps                         # container running
docker ps -a                      # tutti i container
docker images                     # immagini locali
docker logs <container>           # log
docker exec -it <container> sh    # shell dentro container
docker compose up -d              # tutti i servizi
docker compose down               # spegni
docker compose logs -f            # follow log
docker system prune -a            # cleanup totale
```

### npm essenziali

```bash
npm init -y                       # inizializza progetto
npm install <pkg>                 # installa dipendenza runtime
npm install --save-dev <pkg>      # installa dev dependency
npm ci                            # clean install (CI)
npm test                          # esegui test
npm run <script>                  # esegui script
npm audit                         # audit security
npm outdated                      # lista pacchetti obsoleti
```

### gh (GitHub CLI)

```bash
gh repo create                    # crea repo
gh pr create --fill               # crea PR
gh pr list                        # lista PR
gh pr checkout <num>              # checkout di una PR
gh issue list                     # lista issue
gh release create v1.0.0          # crea release
```

## 7.3 — Risorse esterne consigliate

### Git e GitHub

- **Pro Git Book**: https://git-scm.com/book/en/v2 (gratis, completo)
- **Learn Git Branching**: https://learngitbranching.js.org/ (interattivo)
- **GitHub Skills**: https://skills.github.com/

### Linux e shell

- **Linux Journey**: https://linuxjourney.com/
- **The Linux Command Line** (William Shotts): https://linuxcommand.org/tlcl.php (libro gratis)

### Docker

- **Docker Get Started**: https://docs.docker.com/get-started/
- **Docker Curriculum**: https://docker-curriculum.com/

### Node.js / TypeScript

- **Node.js docs**: https://nodejs.org/docs/
- **TypeScript Handbook**: https://www.typescriptlang.org/docs/handbook/intro.html

### Testing

- **Test-Driven Development by Example** (Kent Beck) — libro classico.
- **Vitest docs**: https://vitest.dev/

### Sicurezza

- **OWASP Top 10**: https://owasp.org/Top10/
- **Web Security Academy** (PortSwigger): https://portswigger.net/web-security
- **Krebs on Security** (newsletter): https://krebsonsecurity.com/

### DevOps e produzione

- **The DevOps Handbook** (Gene Kim et al.) — libro fondamentale.
- **Site Reliability Engineering** (Google, gratis): https://sre.google/books/
- **Release It!** (Michael Nygard) — design per produzione.

### Architettura e design

- **The Pragmatic Programmer** (Hunt & Thomas) — classico.
- **Clean Architecture** (Robert C. Martin).
- **Designing Data-Intensive Applications** (Martin Kleppmann) — sistemi distribuiti.

### Workflow e produttività

- **Conventional Commits**: https://www.conventionalcommits.org/
- **Keep a Changelog**: https://keepachangelog.com/
- **Semantic Versioning**: https://semver.org/

### Claude Code

- **Anthropic docs**: https://docs.claude.com/
- **everything-claude-code**: https://github.com/affaan-m/everything-claude-code

## 7.4 — Cosa fare quando sei bloccato

### Errore che non capisci

1. Leggi il messaggio. Davvero. Per intero.
2. Cerca il messaggio esatto su Google (con virgolette).
3. Cerca su Stack Overflow e GitHub Issues del tool.
4. Chiedi a Claude Code: copia-incolla l'errore e il contesto.
5. Se persiste, semplifica: riproduci in un caso minimo isolato.

### Test fallisce in modo non chiaro

1. Esegui solo quel test (`vitest run path/to/test`).
2. Aggiungi `console.log` per ispezionare lo stato.
3. Prova a tornare al checkpoint precedente: c'è un commit fra "verde" e "rosso" che ha rotto qualcosa.
4. `git bisect`: ti aiuta a trovare il commit che ha introdotto il bug.

### CI verde locale ma rosso CI

Causa comune: differenza ambiente. Versione runtime, env var, file system case-sensitivity (Linux vs macOS).

1. Confronta versione runtime: `.nvmrc` vs setup CI.
2. Confronta env var del job CI con il tuo `.env`.
3. Su Mac, file system è case-insensitive: `Foo.ts` e `foo.ts` sono lo stesso file. Su Linux no.

### Sei bloccato da > 1 ora

Stop. Prendi una pausa di 15 minuti. Esci dal computer. La pausa è produttiva, l'ostinazione no.

Al ritorno, riformula il problema: scrivi su carta cosa stai cercando di fare, cosa hai provato, cosa è strano. Spesso la sola scrittura risolve.

Se ancora bloccato dopo: chiedi aiuto. Forum, Discord di linguaggio, Reddit (`r/learnprogramming`, `r/<linguaggio>`). Includi minimal reproducible example.

### Senti che il workflow ti rallenta

Non significa che il workflow sia sbagliato. Significa che stai imparando. La velocità arriva con la pratica. Non saltare passaggi per "andare più veloce": è il modo in cui si accumula debito tecnico che ti rallenterà di più dopo.

Se dopo 3-6 mesi un passaggio specifico ti rallenta sistematicamente, allora valuta di adattarlo. Non prima.

---

# Conclusione

Hai ora tutto quello che serve per partire da zero e arrivare a un workflow professionale completo. La guida è densa: non aspettarti di assimilarla in un giorno.

Suggerimento di percorso:

1. **Settimana 1**: leggi Parte 1 in calma. Apri il terminale, prova i comandi base, gioca con Git su un repo di test.
2. **Settimana 2**: setup completo della macchina (Parte 2). Crea un account GitHub se non l'hai. Verifica tutto.
3. **Settimane 3-4**: segui Parte 3 e 4 su un progetto-giocattolo (puoi copiare l'esempio Personal Notes API o inventarne uno tuo). Scrivi davvero il codice. Apri davvero le PR. Mergia davvero.
4. **Settimana 5**: Parte 5. Deploy reale, dominio reale, certificato reale.
5. **Da lì in poi**: Parte 6 diventa la tua routine.

Quando hai la padronanza di tutta la guida, passa al `workflow-solo-professionale.md` come riferimento operativo. Quel documento assume che tu sappia tutto questo.

Buon viaggio.
