# Guida completa al setup professionale di Claude Code su Windows (WSL2 + Ubuntu)

> Versione: maggio 2026 — adatta a Windows 10/11 con WSL2
> Obiettivo: portarti da zero a un setup di Claude Code di livello professionale per programmazione, capendo a fondo ogni passaggio.

---

## Indice

1. [Cos'è Claude Code e perché ti serve](#1-cosè-claude-code-e-perché-ti-serve)
2. [Perché WSL2 e non Windows nativo](#2-perché-wsl2-e-non-windows-nativo)
3. [Prerequisiti](#3-prerequisiti)
4. [Installazione di WSL2 con Ubuntu](#4-installazione-di-wsl2-con-ubuntu)
5. [Configurazione dell'ambiente di sviluppo in Ubuntu](#5-configurazione-dellambiente-di-sviluppo-in-ubuntu)
6. [Installazione di Claude Code](#6-installazione-di-claude-code)
7. [Primo avvio, login e tour dell'interfaccia](#7-primo-avvio-login-e-tour-dellinterfaccia)
8. [Configurazione professionale: `settings.json`](#8-configurazione-professionale-settingsjson)
9. [Memoria persistente: i file `CLAUDE.md`](#9-memoria-persistente-i-file-claudemd)
10. [Integrazione con VS Code](#10-integrazione-con-vs-code)
11. [MCP servers — collegare Claude a strumenti esterni](#11-mcp-servers--collegare-claude-a-strumenti-esterni)
12. [Plugins e marketplace](#12-plugins-e-marketplace)
13. [Slash commands custom](#13-slash-commands-custom)
14. [Sub-agents (agenti specializzati)](#14-sub-agents-agenti-specializzati)
15. [Hooks (automazioni e guardrail)](#15-hooks-automazioni-e-guardrail)
16. [Skills (competenze riutilizzabili)](#16-skills-competenze-riutilizzabili)
17. [Workflow tipici di programmazione](#17-workflow-tipici-di-programmazione)
18. [Best practices e sicurezza](#18-best-practices-e-sicurezza)
19. [Costi, limiti e modelli](#19-costi-limiti-e-modelli)
20. [Troubleshooting](#20-troubleshooting)
21. [Checklist finale del setup professionale](#21-checklist-finale-del-setup-professionale)

---

## 1. Cos'è Claude Code e perché ti serve

**Claude Code** è l'agente di programmazione ufficiale di Anthropic che vive nel tuo terminale (e si integra con VS Code, JetBrains, ecc.). A differenza di un'autocomplete come Copilot, Claude Code è un *agente*: non si limita a suggerirti righe di codice, ma esegue azioni concrete sul tuo file system — legge file, modifica codice, lancia comandi shell, esegue test, fa commit, naviga repository — operando in cicli di pianificazione/esecuzione/verifica.

In pratica usi Claude Code per:

- Esplorare e capire codebase grandi che non hai mai visto
- Implementare feature complete partendo da una descrizione in italiano
- Fare refactoring ampi e coerenti su più file
- Debuggare scrivendo, eseguendo e iterando sui test
- Automatizzare code review, security review, scrittura di documentazione
- Eseguire task ripetitivi (generare boilerplate, migrazioni, conversioni di formato)

Tutto questo lo fa stando nel tuo terminale, con accesso reale al filesystem e ai comandi: significa potenza, ma anche responsabilità — che gestiremo con permissions, hooks e guardrail.

## 2. Perché WSL2 e non Windows nativo

Da fine 2025 Anthropic distribuisce un installer nativo Windows, e in teoria Claude Code può girare anche lì. Nella pratica però conviene a quasi tutti usare **WSL2 con Ubuntu**, per ragioni concrete:

- **Compatibilità con tooling Unix**: Claude Code lancia spesso comandi shell pensati per ambienti POSIX (`grep`, `find`, `awk`, `sed`, pipes complesse). Su Windows nativo molti falliscono o si comportano diversamente.
- **Path e separatori**: Linux usa `/`, Windows usa `\`. La maggior parte degli script, dei tool open source e delle MCP gira meglio in un ambiente Unix.
- **Permessi file e symlink**: WSL2 supporta correttamente permessi POSIX e symlink, fondamentali per Git, Node, Python, Docker.
- **Coerenza con la produzione**: il 90% dei server di produzione gira Linux. Sviluppare in WSL2 elimina il classico "funziona da me ma non in CI/CD".
- **Performance reali**: WSL2 esegue un kernel Linux completo in una VM leggera; le performance su filesystem `/home/...` sono ottime.
- **Ecosistema MCP**: la maggior parte degli MCP server e dei plugin più maturi è testata principalmente su Linux/macOS.

WSL1 esiste ancora ma è un livello di compatibilità che traduce le syscall Linux in Windows: sconsigliato. Useremo **esclusivamente WSL2**.

## 3. Prerequisiti

### Hardware

- **CPU x86_64 o ARM64** con virtualizzazione attiva nel BIOS/UEFI (cerca "Intel VT-x", "AMD-V" o "SVM Mode" e abilitala se è disabilitata)
- **8 GB di RAM minimi**, 16 GB consigliati (Claude Code + browser + IDE + WSL fanno la loro parte)
- **20 GB di spazio libero** su disco per Ubuntu, Node, repository, modelli di tooling

### Software

- **Windows 10 versione 2004+ (build 19041+)** o **Windows 11** (consigliato): vai in `Impostazioni → Sistema → Informazioni` e controlla la build
- **Windows Terminal** (ottimo, lo installiamo dal Microsoft Store): è il terminale moderno di Microsoft con tab, profili, supporto Unicode pieno
- **Editor**: VS Code Windows (lo integreremo con WSL via la Remote Development extension)

### Account

Claude Code richiede uno di questi:

- Un piano **Claude Pro** (\$20/mese) o **Claude Max** (\$100/200 al mese): ti dà accesso a Claude Code senza pagare a consumo, fino ai limiti del piano. Per programmazione full-time consigliato il Max.
- Oppure **API key Anthropic** con credito a consumo (paghi per token consumati). Più costoso per uso intensivo, ma utile per progetti aziendali con billing separato.

> Suggerimento: se programmi spesso, parti con Claude Max. Eviti l'ansia da contatore e i piani piatti rendono in pochi giorni di uso intenso.

## 4. Installazione di WSL2 con Ubuntu

### 4.1 Abilitare WSL2

Apri **PowerShell come amministratore** (`Win` → digita "PowerShell" → tasto destro → "Esegui come amministratore") ed esegui:

```powershell
wsl --install
```

Questo singolo comando, sui sistemi recenti, fa quattro cose insieme:

1. Abilita la feature "Virtual Machine Platform" di Windows
2. Abilita la feature "Windows Subsystem for Linux"
3. Scarica e installa il kernel Linux di WSL2
4. Installa Ubuntu come distribuzione di default

**Dopo il reboot** (richiesto), riapri PowerShell e verifica che WSL2 sia il default:

```powershell
wsl --set-default-version 2
wsl --status
```

Dovresti vedere `Default Version: 2` e il kernel installato.

### 4.2 Primo avvio di Ubuntu

Cerca "Ubuntu" nel menu Start e aprilo. Al primo avvio:

1. Ti chiede di creare un **username Linux** (minuscolo, senza spazi, ad esempio `user`). Questo è separato dal tuo account Windows.
2. Ti chiede una **password Linux**. Questa serve solo per `sudo` dentro Ubuntu — tienila semplice ma memorabile, non viene sincronizzata da nessuna parte.

Dopo qualche secondo sei nella shell Linux: `user@DESKTOP-XXX:~$`.

### 4.3 Aggiornare il sistema

La prima cosa da fare in qualsiasi sistema Linux fresco è aggiornare i pacchetti:

```bash
sudo apt update && sudo apt upgrade -y
```

- `apt update` aggiorna l'**indice** dei pacchetti (la lista di cosa è disponibile e in che versione)
- `apt upgrade -y` installa effettivamente gli aggiornamenti (`-y` risponde "yes" automaticamente)

### 4.4 Installare Windows Terminal (se non l'hai già)

Apri Microsoft Store, cerca **Windows Terminal** e installalo. Poi aprilo, va su `Impostazioni → Profilo predefinito` e seleziona **Ubuntu**. Da ora ogni nuova finestra di Windows Terminal aprirà direttamente Ubuntu.

### 4.5 Filesystem: dove salvare il codice

Regola d'oro: **lavora sempre dentro `/home/<utente>/`** (cioè `~`), mai su `/mnt/c/...`. Il filesystem Linux nativo di WSL2 è velocissimo; il mount di Windows (`/mnt/c`) è lento per operazioni come `npm install` o `git status` su repo grandi.

Per accedere ai file di WSL da Windows Explorer, scrivi nella barra degli indirizzi: `\\wsl$\Ubuntu\home\<utente>\`.

## 5. Configurazione dell'ambiente di sviluppo in Ubuntu

Questa sezione costruisce la base professionale: shell, build tools, runtime, Git, SSH. Qualsiasi setup serio passa da qui.

### 5.1 Tool essenziali di sviluppo

```bash
sudo apt install -y \
  build-essential \
  curl wget \
  git \
  unzip \
  ca-certificates \
  software-properties-common \
  jq \
  ripgrep \
  fd-find \
  htop \
  tree
```

Cosa stai installando e perché:

- `build-essential`: GCC, make e header standard. Necessari per compilare moduli nativi di Node, Python, ecc.
- `curl` / `wget`: scaricare file da URL. Tutti gli installer di tool moderni li usano.
- `git`: il version control. Lo configureremo a parte.
- `jq`: parser JSON da riga di comando. Indispensabile per ispezionare risposte API e file di configurazione.
- `ripgrep` (`rg`): grep moderno, ordini di grandezza più veloce. Anche Claude Code lo usa internamente.
- `fd-find` (`fdfind`): alternativa moderna a `find`, sintassi più sana.
- `htop` / `tree`: monitor di sistema e visualizzazione directory.

### 5.2 Shell moderna: Zsh + Oh My Zsh (opzionale ma consigliato)

Bash è ottima, ma Zsh con Oh My Zsh ti dà autocompletamento intelligente, history condivisa, prompt informativi e mille piccoli quality-of-life. Se vuoi restare con Bash salta a 5.3.

```bash
sudo apt install -y zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

L'installer ti chiederà se vuoi rendere zsh shell di default: rispondi sì.

Plugin utili (modifica `~/.zshrc`, riga `plugins=(...)`):

```bash
plugins=(git docker docker-compose npm node python pip kubectl)
```

Ricarica con `source ~/.zshrc`.

### 5.3 Node.js via nvm (necessario per Claude Code)

Claude Code richiede **Node.js 18 o superiore**. Da [docs ufficiali](https://code.claude.com/docs/en/setup): non installare Node con `sudo apt install nodejs` (versione vecchia) e **non usare mai `sudo npm install -g`** (causa problemi di permessi e di sicurezza).

Lo strumento corretto è **nvm** (Node Version Manager): ti permette di avere più versioni di Node affiancate e di switchare al volo tra progetti.

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
```

Chiudi e riapri il terminale (oppure `source ~/.bashrc` / `source ~/.zshrc`). Verifica:

```bash
command -v nvm   # deve stampare "nvm"
```

Installa l'ultima LTS di Node e impostala come default:

```bash
nvm install --lts
nvm alias default 'lts/*'
node --version    # es. v22.x.x
npm --version     # es. 10.x.x
```

### 5.4 Configurare Git

Ti identifichi con Git una volta sola, globalmente:

```bash
git config --global user.name "Tuo Nome"
git config --global user.email "Tua Email"
git config --global init.defaultBranch main
git config --global pull.rebase false
git config --global core.editor "code --wait"   # apre VS Code per i commit message
git config --global color.ui auto
```

> Nota su `user.email`: se contribuisci a progetti pubblici e vuoi proteggere la tua mail, usa l'indirizzo "no-reply" che GitHub ti fornisce in `Settings → Emails` (`12345+username@users.noreply.github.com`).

### 5.5 Chiavi SSH per GitHub

Le chiavi SSH ti permettono di clonare e pushare verso GitHub senza inserire password ogni volta, ed è il metodo standard professionale.

Genera una chiave Ed25519 (più moderna e sicura di RSA):

```bash
ssh-keygen -t ed25519 -C "Tua Email"
```

Premi Enter per accettare il path di default (`~/.ssh/id_ed25519`). La passphrase è opzionale ma consigliata: sblocca la chiave una volta a sessione tramite `ssh-agent`.

Avvia l'agent e aggiungi la chiave:

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

Stampa la chiave pubblica e copiala:

```bash
cat ~/.ssh/id_ed25519.pub
```

Vai su [github.com/settings/keys](https://github.com/settings/keys) → **New SSH key** → incolla. Testa con:

```bash
ssh -T git@github.com
```

Dovresti vedere: `Hi <username>! You've successfully authenticated...`.

Per non riavviare l'agent ogni volta, aggiungi in fondo a `~/.zshrc` (o `~/.bashrc`):

```bash
# Avvia ssh-agent solo se non già attivo
if ! pgrep -u "$USER" ssh-agent > /dev/null; then
    eval "$(ssh-agent -s)" > /dev/null
fi
ssh-add -q ~/.ssh/id_ed25519 2>/dev/null
```

### 5.6 Toolchain per più linguaggi (panoramica generica)

Setup minimo per coprire i linguaggi più comuni in modo pulito:

**Python con `pyenv` (multi-versione):**

```bash
sudo apt install -y libssl-dev zlib1g-dev libbz2-dev libreadline-dev \
  libsqlite3-dev libncursesw5-dev xz-utils tk-dev libxml2-dev \
  libxmlsec1-dev libffi-dev liblzma-dev
curl https://pyenv.run | bash
```

Aggiungi a `~/.zshrc` o `~/.bashrc`:

```bash
export PYENV_ROOT="$HOME/.pyenv"
[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"
```

Poi:

```bash
pyenv install 3.12
pyenv global 3.12
pip install --upgrade pip pipx
```

**Go**:

```bash
sudo rm -rf /usr/local/go
wget https://go.dev/dl/go1.22.3.linux-amd64.tar.gz -O /tmp/go.tgz
sudo tar -C /usr/local -xzf /tmp/go.tgz
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.zshrc
```

**Rust**:

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source "$HOME/.cargo/env"
```

**Docker** (utile ma non strettamente necessario): conviene installare **Docker Desktop su Windows** e abilitare l'integrazione WSL2 nella sua UI. Da Ubuntu poi `docker` funziona nativamente.

## 6. Installazione di Claude Code

A questo punto l'ambiente è pronto. Installiamo Claude Code via npm (il metodo più portabile e flessibile, valido su qualsiasi distribuzione):

```bash
npm install -g @anthropic-ai/claude-code
```

> **Nota importante**: la documentazione ufficiale segnala che l'installazione npm sta diventando "deprecata" in favore di un installer nativo, ma rimane pienamente funzionante e oggi (2026) è ancora il metodo più diffuso per WSL2. Quando l'installer nativo Linux sarà standard, il comando di switch sarà documentato in `claude doctor`.

Verifica:

```bash
claude --version
which claude
```

Dovresti vedere il path dentro `~/.nvm/versions/node/.../bin/claude`. Se il comando non viene trovato, controlla che nvm sia caricato (`source ~/.nvm/nvm.sh`).

> **MAI** `sudo npm install -g`. È un anti-pattern noto: rompe i permessi e introduce rischi. Se ottieni errori EACCES, è quasi sempre perché npm non è configurato per installare in `~/.nvm/...` (cosa che nvm fa automaticamente). Verifica con `npm config get prefix`.

## 7. Primo avvio, login e tour dell'interfaccia

Posizionati in una directory di un tuo progetto (anche solo una cartella vuota di test):

```bash
mkdir -p ~/code/test-claude && cd ~/code/test-claude
git init
echo "# Test" > README.md
claude
```

Al primo avvio Claude Code apre un browser per il login OAuth contro il tuo account Anthropic (o ti chiede l'API key, se preferisci). Completa il login: la sessione viene salvata in `~/.claude/`.

Adesso sei nell'interfaccia interattiva. Ecco i comandi essenziali da conoscere subito:

| Comando / Tasto | Cosa fa |
|---|---|
| `<scrivi e Invio>` | Manda un messaggio a Claude |
| `Shift+Invio` | Nuova riga senza inviare |
| `/help` | Mostra tutti gli slash command disponibili |
| `/init` | Genera un `CLAUDE.md` di progetto a partire dal codice esistente |
| `/clear` | Pulisce il contesto della conversazione (utile a metà giornata) |
| `/compact` | Riassume la conversazione mantenendo il contesto chiave |
| `/model` | Cambia il modello (Sonnet, Opus, Haiku) |
| `/permissions` | Gestisce i permessi (cosa Claude può fare senza chiederti) |
| `/agents` | Gestisce sub-agents |
| `/mcp` | Mostra e gestisce gli MCP server connessi |
| `/plugin` | Gestisce plugin |
| `/cost` | Mostra il costo della sessione corrente |
| `Esc` | Interrompe Claude mentre sta lavorando |
| `Ctrl+C` | Esci da Claude Code |

### Modalità operative

Claude Code ha tre modalità principali, scelte caso per caso:

- **Interactive (default)**: chatti, Claude propone azioni e tu approvi.
- **`--print` / `-p`** (non interattiva): utile per scripting CI/CD: `claude -p "spiega questo file" < file.go`.
- **Plan mode** (`Shift+Tab` due volte): Claude pianifica senza eseguire modifiche; ottima per task ampi.

## 8. Configurazione professionale: `settings.json`

Il file di configurazione principale è `~/.claude/settings.json` (utente) o `<progetto>/.claude/settings.json` (progetto). Quello di progetto vince sulle stesse chiavi di quello utente.

Crealo se non esiste:

```bash
mkdir -p ~/.claude
touch ~/.claude/settings.json
```

Ecco un punto di partenza serio per uso da programmatore:

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "model": "claude-sonnet-4-6",
  "theme": "dark",
  "editor": {
    "command": "code",
    "args": ["--wait"]
  },
  "env": {
    "EDITOR": "code --wait",
    "PAGER": "less"
  },
  "permissions": {
    "allow": [
      "Bash(git status)",
      "Bash(git diff:*)",
      "Bash(git log:*)",
      "Bash(git show:*)",
      "Bash(npm test:*)",
      "Bash(npm run *)",
      "Bash(pytest:*)",
      "Bash(go test:*)",
      "Bash(cargo test:*)",
      "Bash(rg:*)",
      "Bash(ls:*)",
      "Bash(cat:*)",
      "Bash(head:*)",
      "Bash(tail:*)",
      "Read(./**)",
      "Edit(./**)"
    ],
    "deny": [
      "Bash(rm -rf /*)",
      "Bash(sudo:*)",
      "Bash(curl:* | sh)",
      "Bash(wget:* | sh)",
      "Bash(git push --force:*)",
      "Bash(git reset --hard:*)"
    ],
    "ask": [
      "Bash(git push:*)",
      "Bash(git commit:*)",
      "Bash(npm publish:*)",
      "Write(/etc/**)",
      "Write(~/**)"
    ]
  },
  "telemetry": false
}
```

Cosa fanno i pattern di permessi:

- **`allow`**: comandi/azioni che Claude può eseguire senza chiederti. Usa wildcard (`:*` significa "qualsiasi argomento").
- **`deny`**: blocchi assoluti. Anche se Claude lo prova, fallisce.
- **`ask`**: ti viene mostrato un prompt di conferma a runtime.

> **Filosofia**: parti restrittivo, allarga col tempo. È più produttivo concedere `Read(./**)` e `Edit(./**)` (lettura/scrittura nella directory del progetto) ma chiedere conferma per `git push` e `git commit`. Per `Bash` privilegia liste positive specifiche per i comandi di test/lint che usi spesso.

Permessi a livello di progetto: copia un file `.claude/settings.json` simile dentro la repo, sovrascrivendo solo le chiavi specifiche. Va in commit insieme al codice (è un asset di team).

## 9. Memoria persistente: i file `CLAUDE.md`

I file `CLAUDE.md` sono la **memoria persistente** di Claude Code. Vengono caricati automaticamente all'avvio e dicono a Claude *come* lavorare nel tuo contesto. Sono il singolo intervento con maggior ROI per la qualità delle risposte.

Esistono tre livelli, caricati in ordine (i più specifici sovrascrivono i generici):

1. **Globale utente**: `~/.claude/CLAUDE.md` — tue preferenze valide ovunque
2. **Progetto**: `<repo>/CLAUDE.md` — convenzioni del progetto
3. **Sotto-cartella**: `<repo>/<dir>/CLAUDE.md` — regole specifiche di un modulo

### 9.1 Esempio di `~/.claude/CLAUDE.md` (globale)

```markdown
# Preferenze utente

## Lingua e stile
- Rispondi sempre in italiano salvo richiesta esplicita.
- Sii conciso. No emoji nei commit né nel codice.
- Quando spieghi qualcosa di tecnico, vai dritto al punto, poi approfondisci.

## Stile di codice (default trasversale)
- Usa nomi descrittivi, evita abbreviazioni criptiche.
- Funzioni piccole, single responsibility.
- Commenti solo per spiegare *perché*, non *cosa*.
- Niente codice morto, niente `console.log` lasciati nel codice committato.

## Workflow Git
- Branch convention: `tipo/breve-descrizione` (`feat/login-oauth`, `fix/null-pointer-cart`)
- Conventional Commits: `feat:`, `fix:`, `refactor:`, `chore:`, `docs:`, `test:`
- Mai force-push su `main` o `develop`. Su feature branch va bene `--force-with-lease`.

## Test
- Aggiungi sempre test quando modifichi logica di business.
- Per bug-fix: prima scrivi un test che fallisce, poi correggi.
```

### 9.2 Esempio di `CLAUDE.md` di progetto

```markdown
# Progetto: <nome>

## Stack
- Linguaggio: TypeScript + Node 22
- Framework: Next.js 15 (App Router)
- DB: PostgreSQL + Prisma
- Test: Vitest + Playwright

## Convenzioni
- Path alias: `@/` punta a `src/`
- API in `src/app/api/<route>/route.ts`
- Componenti server di default; `"use client"` solo dove serve davvero
- Stati globali: Zustand store in `src/stores/`

## Comandi importanti
- `pnpm dev`: avvia il dev server
- `pnpm test`: unit + integration
- `pnpm test:e2e`: end-to-end
- `pnpm db:migrate`: applica migrazioni Prisma

## Cose da NON fare
- Non importare direttamente da `node_modules` — usa solo gli alias
- Non scrivere SQL grezzo, sempre via Prisma
- Non usare `any` salvo casi documentati
```

### 9.3 Generare `CLAUDE.md` automaticamente

Dentro un progetto esistente:

```
> /init
```

Claude scansiona il codice e genera un primo `CLAUDE.md` ragionevole. Editalo manualmente per affinarlo: la versione finale dovrebbe essere **breve, specifica, prescrittiva**. Evita la tentazione di scrivere romanzi: ogni riga in `CLAUDE.md` consuma contesto in ogni messaggio.

## 10. Integrazione con VS Code

VS Code è il setup più diffuso e Claude Code ha integrazione di prima classe.

### 10.1 Estensioni necessarie

Installa su VS Code Windows (non dentro WSL):

- **WSL** (Microsoft) — apre VS Code in modalità remote dentro WSL2
- **Claude Code for VS Code** (Anthropic) — integra Claude Code con l'editor

Apri Ubuntu, vai nel progetto e lancia:

```bash
code .
```

VS Code si avvia in **WSL Remote**: l'editor è in Windows, ma il filesystem, il terminale integrato, le estensioni di linguaggio e Claude Code stesso girano in WSL2. È il setup giusto.

### 10.2 Cosa cambia con l'integrazione

Con l'estensione Claude Code attiva:

- Apri un nuovo riquadro Claude direttamente in VS Code (`Ctrl+Esc` di default)
- Le selezioni e i file aperti vengono passati a Claude come contesto
- Le diff proposte si vedono direttamente nell'editor con accept/reject visivi
- Il terminale integrato condivide la sessione di Claude

Da terminale puoi sempre lanciare `claude` come al solito: i due flussi convivono.

### 10.3 Estensioni di sviluppo consigliate (in WSL)

Installa queste *dentro* il profilo WSL di VS Code (l'icona dell'estensione mostra "Install in WSL: Ubuntu"):

- ESLint, Prettier (JS/TS)
- Python, Pylance, Ruff
- GitLens
- Error Lens
- EditorConfig
- Docker
- Database Client / Prisma (se ti serve)

## 11. MCP servers — collegare Claude a strumenti esterni

**MCP (Model Context Protocol)** è uno standard aperto di Anthropic per estendere il contesto di Claude. Un MCP server è un processo che espone *tool* (funzioni invocabili) e *resources* (dati leggibili) tramite un protocollo documentato. Claude Code ne è client: una volta configurato un MCP server, Claude ha accesso ai suoi tool come se fossero nativi.

> **Mental model**: un MCP è "una libreria di funzioni che Claude può chiamare". Esempi: `github` espone `create_issue`, `get_pull_request`, ecc. `filesystem` espone `read_file`, `list_directory`. `postgres` espone `query`, `list_tables`. E così via.

### 11.1 MCP utili per programmazione

| MCP | A cosa serve |
|---|---|
| `filesystem` | Lettura/scrittura file in directory specifiche (utile come complemento ai tool nativi) |
| `git` | Tool Git ad alto livello (status, log, diff, blame) |
| `github` | API GitHub (issue, PR, commit, code search) |
| `playwright` | Automazione browser per testing E2E |
| `postgres` / `sqlite` | Query e ispezione DB |
| `context7` | Documentazione live di librerie ad alta qualità |
| `sequential-thinking` | Tool di "ragionamento step-by-step" guidato |
| `fetch` | Fetch di URL HTTP, comodo per API non gestite altrove |

### 11.2 Aggiungere un MCP server

Modo veloce, da CLI:

```bash
claude mcp add github -- npx -y @modelcontextprotocol/server-github
```

Per molti MCP server serve una variabile d'ambiente (es. token GitHub):

```bash
claude mcp add github \
  -e GITHUB_PERSONAL_ACCESS_TOKEN=ghp_xxx \
  -- npx -y @modelcontextprotocol/server-github
```

Il file viene aggiornato in `~/.claude.json` (lato MCP). Per vedere quelli attivi: `/mcp` dentro Claude Code, oppure:

```bash
claude mcp list
```

### 11.3 Scope: utente, progetto, locale

Quando aggiungi un MCP, scegli lo scope con `--scope`:

- `--scope user` (default): valido per te, ovunque
- `--scope project`: file `.mcp.json` nel progetto, va in commit, valido per il team
- `--scope local`: solo per te, solo questo progetto

Per progetti di team usa **`project`** in modo che tutti abbiano gli stessi MCP.

### 11.4 Esempio reale: GitHub MCP

1. Crea un Personal Access Token su [github.com/settings/tokens](https://github.com/settings/tokens) con scope `repo`, `read:org`, `workflow`.
2. Aggiungi:
   ```bash
   claude mcp add github \
     -e GITHUB_PERSONAL_ACCESS_TOKEN=ghp_xxxxxxxxxxxx \
     --scope user \
     -- npx -y @modelcontextprotocol/server-github
   ```
3. In Claude Code: `> Apri la PR #42 e dimmi cosa cambia rispetto a main`. Claude userà i tool dell'MCP GitHub.

> **Sicurezza**: i token finiscono in chiaro nel file di configurazione. Non committare mai `~/.claude.json`. Per condividere config di progetto, usa `.mcp.json` con riferimenti a variabili d'ambiente (`${env:GITHUB_TOKEN}`).

## 12. Plugins e marketplace

Dal 2026 Claude Code supporta **plugins**: pacchetti che racchiudono in un solo install più cose insieme — slash commands, sub-agents, MCP servers, hooks, skills. Sono il modo migliore per condividere setup tra macchine e tra colleghi.

### 12.1 Installare un plugin

```
> /plugin
```

Si apre la UI: cerchi un plugin nei marketplace registrati, lo selezioni e si installa. Alternativamente:

```
> /plugin install <nome-plugin>
```

### 12.2 Aggiungere un marketplace

Un marketplace è un repository (locale o Git) che elenca plugin:

```
> /plugin marketplace add anthropics/claude-code
> /plugin marketplace add user/claude-marketplace
```

### 12.3 Plugin utili da installare subito

- **anthropic-skills**: la collezione ufficiale di skill per docx/pptx/xlsx/pdf, schedulazioni, ecc.
- **github-workflow**: review automatica di PR, check di CI, ecc.
- **security-review**: code-audit pre-merge

> Ogni plugin che installi può aggiungere tool e potenzialmente eseguire codice. Installa solo da fonti fidate, leggi il `README.md` e il file `plugin.json` prima.

## 13. Slash commands custom

Gli slash command sono prompt riutilizzabili. Esempio: `/review`, `/init`, `/security-review` sono slash command. Puoi crearne dei tuoi.

Posiziona file `.md` in:

- `~/.claude/commands/` per comandi globali
- `<progetto>/.claude/commands/` per comandi di progetto (versione-controllabili)

Esempio — `~/.claude/commands/test-cov.md`:

```markdown
---
description: Lancia i test e mostra la coverage delle aree modificate
allowed-tools: Bash
---

Esegui i test del progetto corrente cercando di rilevare automaticamente lo stack
(npm test, pytest, go test, cargo test). Dopo l'esecuzione:

1. Mostra solo i fallimenti (se ci sono)
2. Calcola la coverage delle righe modificate vs main
3. Suggerisci 3 test mancanti basati sui diff non coperti
```

Da Claude Code: `/test-cov`. Il front-matter YAML è opzionale ma utile per restringere i tool disponibili.

### Argomenti

I command supportano argomenti via `$1`, `$2`, … o `$ARGUMENTS`:

```markdown
---
description: Spiega un file specifico
---

Apri il file $1 e spiegamelo come a un nuovo membro del team:
struttura, dipendenze chiave, e dove finisce nel sistema più grande.
```

Uso: `/explain src/auth/login.ts`.

## 14. Sub-agents (agenti specializzati)

I **sub-agent** sono agenti specializzati con un loro prompt, un loro toolset e un loro contesto separato. Servono a delegare task verticali (fare ricerca, scrivere test, fare code review) senza inquinare la conversazione principale.

### 14.1 Quando usarli

- Task ripetitivi e ben definiti (ricerche di pattern, bump di dipendenze)
- Operazioni che richiedono molto contesto temporaneo che poi puoi buttare
- Funzioni con permessi *diversi* dal main (es. un agente solo-lettura)

### 14.2 Definire un sub-agent

File `.md` in:

- `~/.claude/agents/` (utente)
- `<progetto>/.claude/agents/` (progetto)

Esempio — `~/.claude/agents/test-writer.md`:

```markdown
---
name: test-writer
description: Specializzato nella scrittura di unit test di alta qualità. Da usare quando l'utente chiede test per una funzione o una classe.
tools: Read, Edit, Write, Bash, Grep, Glob
model: sonnet
---

Sei un ingegnere software senior specializzato in test automatici.
Quando ricevi una funzione o un modulo:

1. Identifica le condizioni di confine (input vuoti, valori limite, errori)
2. Scrivi test deterministici, non flaky, indipendenti
3. Usa il framework già presente nel progetto (rilevalo da `package.json`/`pyproject.toml`/`go.mod`)
4. Naming convention: `nomeFunzione_dovrebbeFareX_quandoY`
5. Aggiungi sempre almeno un test del happy path, uno di errore, e uno di edge case
6. Mai mock superflui: mocca solo I/O esterni reali

Restituisci solo il codice dei test, pronto da eseguire.
```

Lanciatlo con: `> Usa il sub-agent test-writer per scrivere test su src/utils/parser.ts`.

### 14.3 Limitare i tool

Per agent solo-lettura (utili per audit), restringi:

```yaml
tools: Read, Grep, Glob
```

Senza `Bash`, `Write`, `Edit` quel sub-agent non può modificare nulla per design.

## 15. Hooks (automazioni e guardrail)

Gli hook sono **eventi del ciclo di vita** di Claude Code a cui puoi attaccare comandi shell o prompt. Servono a due scopi:

1. **Automazione**: formatta il codice dopo ogni modifica, lancia il linter, aggiorna un'index, ecc.
2. **Guardrail**: blocca azioni rischiose prima che vengano eseguite.

I tipi più usati:

- `PreToolUse`: prima di eseguire un tool. Può bloccare l'azione.
- `PostToolUse`: dopo un tool. Tipico per formatter/linter.
- `UserPromptSubmit`: ogni volta che mandi un messaggio.
- `SessionStart`: all'avvio della sessione.

### 15.1 Esempio: auto-format dopo ogni edit

In `~/.claude/settings.json` (o per progetto):

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "if [ -f \"$CLAUDE_FILE_PATH\" ]; then case \"$CLAUDE_FILE_PATH\" in *.ts|*.tsx|*.js|*.jsx) npx prettier --write \"$CLAUDE_FILE_PATH\" ;; *.py) ruff format \"$CLAUDE_FILE_PATH\" ;; *.go) gofmt -w \"$CLAUDE_FILE_PATH\" ;; *.rs) rustfmt \"$CLAUDE_FILE_PATH\" ;; esac; fi"
          }
        ]
      }
    ]
  }
}
```

Cosa fa: dopo ogni `Edit` o `Write`, se il file ha estensione conosciuta lancia il formatter giusto. Pulisce automaticamente lo stile, indipendentemente da Claude.

### 15.2 Esempio: bloccare comandi pericolosi

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "if echo \"$CLAUDE_TOOL_INPUT\" | grep -Eq 'rm -rf / |dd if=|mkfs|:(){:|:&};:'; then echo 'BLOCKED: comando potenzialmente distruttivo'; exit 2; fi"
          }
        ]
      }
    ]
  }
}
```

L'`exit 2` interrompe l'esecuzione e mostra il messaggio a Claude.

### 15.3 Esempio: notifica desktop a fine task

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "powershell.exe -Command \"New-BurntToastNotification -Text 'Claude Code', 'Task completato'\""
          }
        ]
      }
    ]
  }
}
```

(Da WSL2 puoi chiamare comandi Windows con `.exe`.)

## 16. Skills (competenze riutilizzabili)

Le **skill** sono unità di conoscenza specializzata che Claude carica on-demand quando il contesto le richiede. A differenza degli slash command, non le invochi tu: le invoca Claude quando un trigger nella descrizione fa match con la richiesta dell'utente.

Una skill è una directory con un `SKILL.md` che dichiara *quando* usarla e *come*.

```
~/.claude/skills/
└── react-best-practices/
    ├── SKILL.md
    ├── examples/
    └── references.md
```

Esempio — `SKILL.md`:

```markdown
---
name: react-best-practices
description: Utilizza questa skill quando l'utente lavora su componenti React, hook, performance o pattern moderni di React 19+.
---

# React best practices

## Quando applicare
- Composizione di componenti
- Uso corretto di Server Components vs Client Components
- Pattern di data fetching (Suspense, RSC)

## Linee guida
1. Preferire Server Components di default; "use client" solo per stato/effects.
2. ...
```

Le skill ufficiali (`docx`, `pptx`, `xlsx`, `pdf`) si installano via plugin **anthropic-skills**.

## 17. Workflow tipici di programmazione

Pochi pattern coprono il 90% dell'uso quotidiano. Tienili a mente.

### 17.1 Esplorare un progetto nuovo

```
> /init
> Leggi CLAUDE.md, poi guarda la struttura del repo e dimmi: 
  qual è il punto di ingresso, come vengono routati i moduli, 
  e dove sta la logica di autenticazione.
```

### 17.2 Implementare una feature

Pattern in tre fasi: **plan → diff → review**.

```
> Devo aggiungere un endpoint POST /api/users che:
  - valida email e password (zod)
  - hash password con argon2
  - salva su DB con Prisma
  - restituisce un JWT
  
  Prima fammi solo un piano (Shift+Tab per plan mode).
```

Approvi il piano. Poi:

```
> Procedi con l'implementazione.
```

A fine task:

```
> Mostrami il diff complessivo. Spiegami i punti dove hai dovuto fare scelte di design.
```

### 17.3 Debug guidato

```
> Quando lancio `pnpm test src/auth.test.ts` ottengo:
  [incolla output]
  
  Investiga: trova la causa root, non lavorare sui sintomi. 
  Aggiungi un test che riproduce il bug, poi correggi.
```

### 17.4 Refactor su larga scala

```
> Rinomina ovunque `getUser` in `findUserById`. Usa il sub-agent Explore 
  per trovare tutti i riferimenti, poi fai un commit per ogni file modificato.
```

### 17.5 Code review locale prima del push

```
> /review
```

Oppure custom:

```
> Prima di pushare, fammi una review onesta sul mio diff vs main:
  - smell di codice
  - test mancanti
  - rischi runtime (null, race condition, sicurezza)
```

### 17.6 Scrittura di test

```
> Usa il sub-agent test-writer per coprire src/services/payment.ts. 
  Voglio almeno un test di happy path, uno di errore di rete e uno di input invalido.
```

## 18. Best practices e sicurezza

### 18.1 Igiene del contesto

- Una conversazione → un task. A task finito, `/clear`.
- Per task lunghi, `/compact` periodicamente.
- Non incollare interi file da 2000 righe: meglio dare il path e lasciare che Claude legga.

### 18.2 Permessi: principio del privilegio minimo

Concedi `allow` con generosità per comandi *idempotenti e di lettura* (`git status`, `rg`, `cat`). Tieni in `ask` tutto ciò che è *irreversibile o esterno* (`git push`, `npm publish`, deploy). Tieni in `deny` ciò che non vorrai mai (`sudo`, `curl ... | sh`).

### 18.3 Non committare segreti

Aggiungi a `.gitignore` di ogni progetto:

```
.claude/settings.local.json
.env
.env.*
!.env.example
```

E al tuo `~/.gitconfig`:

```bash
git config --global core.excludesFile ~/.gitignore_global
```

In `~/.gitignore_global`:

```
.claude/settings.local.json
.DS_Store
```

### 18.4 Branch dedicato per task agentici

Per task ampi guidati da Claude, lavora su un branch isolato:

```bash
git switch -c agent/refactor-payments
```

Così se qualcosa va storto, è sempre `git switch main` per tornare in stato pulito.

### 18.5 Verifica, non fidarti

Ogni diff proposta da Claude va letta. Ogni test va eseguito. Per task critici, esci da Claude e ri-leggi il codice modificato come fosse di un collega: spesso noti pattern non ottimali che Claude può aver introdotto seguendo strutture esistenti.

### 18.6 Sicurezza nei prompt

Mai incollare in chat:

- Password, API key in chiaro
- Dati personali di clienti reali (anonimizza)
- Contenuti di file `.env`

Per testare con dati realistici, usa generatori (Faker, mockoon, ecc.).

## 19. Costi, limiti e modelli

### 19.1 Modelli disponibili (maggio 2026)

- **Claude Opus 4.6** (`claude-opus-4-6`): il più capace, ottimo per refactor complessi, design di sistema, debug difficili. Più lento e più costoso.
- **Claude Sonnet 4.6** (`claude-sonnet-4-6`): default consigliato. Bilanciato per il 90% dei task di coding quotidiani.
- **Claude Haiku 4.5** (`claude-haiku-4-5`): velocissimo, economico, perfetto per task semplici, ricerca codice, sub-agent ad alto volume.

Cambi al volo con `/model`.

### 19.2 Pianificare il modello per task

```
> /model opus      # task complesso (refactor architetturale)
> /model sonnet    # default
> /model haiku     # ricerche, formattazione, task ripetitivi
```

Nel `settings.json` per progetto puoi fissare `claude-sonnet-4-6` come default.

### 19.3 Capire la spesa

```
> /cost
```

Mostra token in input/output e spesa stimata. Su piano Claude Pro/Max non paghi per token ma hai limiti di sessione: `/cost` ti dice quanto contesto hai usato.

## 20. Troubleshooting

### `claude: command not found`

```bash
# nvm probabilmente non è caricato nella shell corrente
source ~/.nvm/nvm.sh
nvm use default
which claude
```

Se persiste: `npm root -g` e verifica che il path sia nel `$PATH`.

### Errori EACCES durante `npm install -g`

Hai installato Node con `apt` o usato `sudo`. Reinstalla via nvm e disinstalla la versione di sistema:

```bash
sudo apt remove nodejs npm
nvm install --lts
```

### Login non si apre nel browser

Da WSL2 potrebbe non riuscire a lanciare il browser di Windows. Soluzioni:

```bash
# 1) Forza l'opener Windows
export BROWSER='/mnt/c/Program Files/Google/Chrome/Application/chrome.exe'
# 2) Oppure copia manualmente l'URL che Claude stampa e aprilo a mano in Chrome
```

### MCP server non si avvia

```bash
claude mcp list                # vedi quali sono attivi
claude mcp get <nome>          # ispeziona la config
claude mcp remove <nome>       # rimuovi e riprova
```

Verifica che il binario richiesto (`npx`, `uvx`) sia installato e raggiungibile.

### Performance lenta su `/mnt/c/...`

Sposta il repo dentro `~/code/...`. WSL2 è veloce solo sul filesystem nativo.

### `claude doctor`

Il comando di diagnostica integrato. Lancialo se qualcosa non torna:

```bash
claude doctor
```

Stampa versioni, percorsi, MCP attivi, eventuali errori di configurazione.

### Reset completo

```bash
rm -rf ~/.claude.json ~/.claude/.credentials*
claude   # ti chiede di rifare login
```

(Non rimuove `settings.json`, `CLAUDE.md`, agents, commands, skills — quelli restano.)

## 21. Checklist finale del setup professionale

Quando hai completato la guida, dovresti avere:

- [x] WSL2 installato con Ubuntu, aggiornato all'ultima versione
- [x] Windows Terminal con profilo default Ubuntu
- [x] Node 22 LTS via nvm (mai `sudo`, mai `apt nodejs`)
- [x] Git configurato con il tuo nome/email reali
- [x] Chiave SSH Ed25519 caricata su GitHub e ssh-agent in autostart
- [x] Pyenv / Go / Rust installati nei toolchain dei linguaggi che usi
- [x] Claude Code installato globalmente, login fatto
- [x] `~/.claude/settings.json` con permissions sensate (allow/ask/deny)
- [x] `~/.claude/CLAUDE.md` globale con preferenze personali
- [x] `CLAUDE.md` in ogni progetto importante (almeno per quello principale)
- [x] VS Code Windows + estensione WSL + estensione Claude Code
- [x] Almeno un MCP server utile installato (es. `github`)
- [x] Plugin `anthropic-skills` installato
- [x] Almeno un slash command custom in `~/.claude/commands/`
- [x] Almeno un sub-agent custom in `~/.claude/agents/` (es. `test-writer`)
- [x] Hook `PostToolUse` per auto-format
- [x] Hook `PreToolUse` per bloccare comandi distruttivi
- [x] `.gitignore` globale con `.claude/settings.local.json` e `.env*`

A questo punto il tuo setup è allineato a quello di un programmatore professionista che usa Claude Code in produzione. La parte "professionale" non è tanto avere installato tutto, ma sapere *quando* invocare cosa: usa la sezione 17 come riferimento per i workflow e cresci da lì, aggiungendo skill e sub-agent man mano che identifichi pattern ricorrenti nel tuo lavoro.

---

## Appendice A — Riferimenti rapidi ai path

| Cosa | Path |
|---|---|
| Settings utente | `~/.claude/settings.json` |
| Memory utente | `~/.claude/CLAUDE.md` |
| Slash commands utente | `~/.claude/commands/*.md` |
| Sub-agents utente | `~/.claude/agents/*.md` |
| Skills utente | `~/.claude/skills/<nome>/SKILL.md` |
| Settings progetto | `<repo>/.claude/settings.json` |
| Memory progetto | `<repo>/CLAUDE.md` |
| MCP progetto | `<repo>/.mcp.json` |
| Credenziali (NON committare) | `~/.claude/.credentials*` |
| File globale di stato MCP | `~/.claude.json` |

## Appendice B — Comandi CLI di Claude Code essenziali

```bash
claude                              # avvia interactive
claude -p "spiega questo file"      # one-shot non interattivo
claude --continue                   # riprende l'ultima sessione
claude --resume                     # mostra menu per riaprire una sessione
claude --model opus                 # avvia con un modello specifico
claude doctor                       # diagnostica
claude mcp list                     # MCP attivi
claude mcp add <nome> -- <cmd>      # aggiungi MCP
claude mcp remove <nome>            # rimuovi MCP
claude config get|set               # ispeziona/setta config a riga di comando
claude --help                       # tutta la lista di flag
```

## Appendice C — Risorse ufficiali

- Documentazione: <https://code.claude.com/docs>
- Setup avanzato: <https://code.claude.com/docs/en/setup>
- Settings: <https://code.claude.com/docs/en/settings>
- Sub-agents: <https://code.claude.com/docs/en/sub-agents>
- MCP overview: <https://modelcontextprotocol.io>
- Plugins di Anthropic: <https://claude.com/blog/claude-code-plugins>

---

> Questa guida è stata pensata come "punto di partenza che dura". Riprendila ogni qualche mese: Claude Code evolve in fretta (skills, plugins, hooks sono cambiati molto nell'ultimo anno) e tornare a verificare le sezioni 11–16 ti tiene allineato alle novità.
