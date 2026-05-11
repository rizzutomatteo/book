# Guida Completa: Applicazione Desktop per Gestione Palestra

**Stack tecnologico:** Tauri + Rust + Svelte + SQLite + GitHub + Claude Code

Questa guida ti accompagnerà passo passo nella creazione di un'applicazione desktop professionale per la gestione di clienti e schede palestra. Anche se sei un principiante, seguendo questa guida lavorerai come un professionista, adottando le best practice del settore.

---

## Indice

1. [Panoramica del Progetto](#1-panoramica-del-progetto)
2. [Perché questo Stack Tecnologico](#2-perché-questo-stack-tecnologico)
3. [Prerequisiti e Setup dell'Ambiente](#3-prerequisiti-e-setup-dellambiente)
4. [Installazione degli Strumenti](#4-installazione-degli-strumenti)
5. [Configurazione di Claude Code](#5-configurazione-di-claude-code)
6. [Creazione del Progetto Tauri + Svelte](#6-creazione-del-progetto-tauri--svelte)
7. [Configurazione di Git e GitHub](#7-configurazione-di-git-e-github)
8. [Architettura dell'Applicazione](#8-architettura-dellapplicazione)
9. [Progettazione del Database SQLite](#9-progettazione-del-database-sqlite)
10. [Sviluppo del Backend Rust](#10-sviluppo-del-backend-rust)
11. [Sviluppo del Frontend Svelte](#11-sviluppo-del-frontend-svelte)
12. [Funzionalità Principali da Implementare](#12-funzionalità-principali-da-implementare)
13. [Testing e Qualità del Codice](#13-testing-e-qualità-del-codice)
14. [Build, Packaging e Distribuzione](#14-build-packaging-e-distribuzione)
15. [Best Practice Professionali](#15-best-practice-professionali)
16. [Risoluzione Problemi Comuni](#16-risoluzione-problemi-comuni)
17. [Roadmap di Apprendimento](#17-roadmap-di-apprendimento)

---

## 1. Panoramica del Progetto

### Cosa costruiremo

Un'applicazione desktop nativa multipiattaforma (Windows, macOS, Linux) chiamata **GymManager** che permette al gestore di una palestra (o personal trainer) di:

- Gestire l'anagrafica dei clienti (dati personali, contatti, foto, note)
- Tracciare abbonamenti e scadenze
- Creare e gestire **schede di allenamento** personalizzate
- Catalogare esercizi con descrizioni, gruppi muscolari e media
- Registrare progressi e misurazioni (peso, BMI, circonferenze)
- Generare report e statistiche
- Esportare schede in PDF da stampare o inviare al cliente

### Caratteristiche tecniche

L'app sarà **offline-first** (funziona senza internet), con database locale SQLite. Sarà installabile come un programma desktop tradizionale (file .exe, .dmg, .deb), ma costruita con tecnologie web moderne per l'interfaccia.

---

## 2. Perché questo Stack Tecnologico

Capire **perché** usiamo determinate tecnologie è fondamentale per crescere come sviluppatore.

### Tauri

Framework che permette di creare applicazioni desktop usando un backend Rust e un frontend web (HTML/CSS/JS). È l'alternativa moderna a Electron: produce eseguibili molto più piccoli (3-10 MB invece di 100+ MB), consuma meno RAM e ha un'architettura di sicurezza più robusta. Usa la webview nativa del sistema operativo invece di includere un intero browser Chromium.

### Rust

Linguaggio di sistema con performance pari a C/C++ ma con sicurezza della memoria garantita a compile-time. Sarà il nostro backend: gestirà database, logica di business, file system e operazioni pesanti. Ha una curva di apprendimento iniziale ripida, ma il compilatore è il miglior insegnante.

### Svelte

Framework frontend che, a differenza di React o Vue, compila il codice in JavaScript vanilla ottimizzato durante la build. Risultato: bundle più piccoli, performance migliori, codice più leggibile. Perfetto per applicazioni desktop dove le risorse contano.

### SQLite

Database relazionale embedded: un singolo file sul disco, zero configurazione, transazioni ACID, ottimo per applicazioni single-user o piccoli team. È il database più diffuso al mondo (presente su ogni smartphone Android e iPhone).

### GitHub

Piattaforma di hosting per repository Git. Ci permetterà di versionare il codice, fare backup automatici, collaborare, gestire issue e bug, e configurare CI/CD per build automatiche.

### Claude Code

Strumento CLI di Anthropic che porta Claude direttamente nel tuo terminale. Sarà il tuo "pair programmer" durante tutto lo sviluppo: ti aiuterà a scrivere codice, fare refactoring, risolvere bug, scrivere test e documentazione. Usare bene Claude Code è oggi una skill professionale a tutti gli effetti.

---

## 3. Prerequisiti e Setup dell'Ambiente

### Hardware consigliato

Un computer con almeno 8 GB di RAM (16 GB consigliati), 10 GB di spazio libero, processore relativamente recente. La compilazione di Rust può essere intensiva.

### Conoscenze di base utili (ma non obbligatorie)

Concetti generali di programmazione (variabili, funzioni, condizionali, cicli), nozioni base di HTML/CSS/JavaScript, idea di cosa sia un database e una query SQL. Se mancano, le imparerai strada facendo: ogni concetto verrà spiegato.

### Sistema operativo

La guida funziona su Windows 10/11, macOS 12+ e principali distribuzioni Linux. Alcuni comandi cambiano leggermente: lo segnaleremo dove necessario.

---

## 4. Installazione degli Strumenti

Installeremo tutti gli strumenti nell'ordine corretto. Ogni passaggio include il comando di verifica.

### 4.1 Editor di codice: Visual Studio Code

Scarica e installa VS Code da https://code.visualstudio.com. È gratuito, open source e supporta perfettamente tutti gli strumenti di questo stack.

Estensioni consigliate da installare (dal pannello Extensions, Ctrl+Shift+X):

- **rust-analyzer** — supporto Rust completo
- **Svelte for VS Code** — sintassi e IntelliSense Svelte
- **Tauri** — snippet e supporto Tauri
- **SQLite Viewer** — visualizzare il database direttamente in VS Code
- **GitLens** — superpoteri Git
- **Error Lens** — errori inline più visibili
- **Prettier** — formattazione automatica
- **EditorConfig** — coerenza tra editor

### 4.2 Git

Scarica da https://git-scm.com. Su macOS arriva con gli Xcode Command Line Tools (`xcode-select --install`). Su Linux usa il package manager (`sudo apt install git` su Debian/Ubuntu).

Configurazione iniziale obbligatoria (sostituisci con i tuoi dati):

```bash
git config --global user.name "Mario Rossi"
git config --global user.email "mario.rossi@example.com"
git config --global init.defaultBranch main
git config --global pull.rebase false
```

Verifica: `git --version`

### 4.3 Node.js e npm

Tauri usa Node.js per gestire il frontend. Installa la versione **LTS** da https://nodejs.org (al momento la 20.x o superiore). Su sistemi Unix consigliamo `nvm` (Node Version Manager) per gestire più versioni.

Verifica: `node --version` e `npm --version`

### 4.4 Rust e Cargo

Rust si installa con **rustup**, il gestore di versioni ufficiale. Su tutti i sistemi:

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Su Windows scarica `rustup-init.exe` da https://rustup.rs.

Dopo l'installazione, riavvia il terminale. Verifica:

```bash
rustc --version
cargo --version
```

`cargo` è il package manager e build tool di Rust, equivalente di npm per JavaScript.

### 4.5 Dipendenze di sistema per Tauri

Tauri ha bisogno di alcune librerie native per compilare. Le istruzioni complete e aggiornate sono su https://tauri.app/start/prerequisites/. In sintesi:

**Windows:** installa **Microsoft C++ Build Tools** (incluse in Visual Studio Build Tools) e **WebView2** (preinstallato su Windows 11).

**macOS:** Xcode Command Line Tools: `xcode-select --install`

**Linux (Debian/Ubuntu):**
```bash
sudo apt update
sudo apt install -y libwebkit2gtk-4.1-dev build-essential curl wget file libxdo-dev libssl-dev libayatana-appindicator3-dev librsvg2-dev
```

### 4.6 Tauri CLI

Una volta che Rust funziona, installa la CLI di Tauri:

```bash
cargo install tauri-cli --version "^2.0"
```

Verifica: `cargo tauri --version`

### 4.7 pnpm (opzionale ma consigliato)

`pnpm` è un'alternativa più veloce ed efficiente a npm. Installa con:

```bash
npm install -g pnpm
```

In questa guida useremo `pnpm`, ma puoi sostituirlo con `npm` in qualsiasi comando.

---

## 5. Configurazione di Claude Code

Claude Code è lo strumento che ti permetterà di sviluppare come un professionista anche se sei alle prime armi. Ti guida, scrive codice insieme a te, spiega cosa fa, e cattura errori che potresti non notare.

### 5.1 Installazione

Claude Code richiede Node.js. Installalo globalmente:

```bash
npm install -g @anthropic-ai/claude-code
```

Verifica: `claude --version`

### 5.2 Autenticazione

Al primo avvio nella tua cartella di progetto:

```bash
claude
```

Verrai guidato attraverso il login (account Claude o API key). Segui le istruzioni sullo schermo.

### 5.3 Come usare Claude Code professionalmente

Claude Code non è un autocomplete: è un collega. Usalo così:

- **Spiegati prima di chiedergli di scrivere codice.** Descrivi cosa vuoi ottenere, perché, e i vincoli. Più contesto fornisci, migliore sarà l'output.
- **Lavora a piccoli passi.** Una task alla volta. "Crea la struttura del database clienti" è meglio di "fai tutto il backend".
- **Leggi il codice che genera.** Non accettare modifiche alla cieca: chiedi spiegazioni sui pezzi che non capisci. È il modo più veloce per imparare.
- **Usa file `CLAUDE.md`.** Nella radice del progetto, crea un file `CLAUDE.md` che descrive l'architettura, le convenzioni di codice e gli obiettivi. Claude lo leggerà automaticamente e adatterà il suo lavoro al tuo contesto.
- **Sfrutta i comandi slash.** `/init` per generare un primo CLAUDE.md, `/review` per fargli rivedere il tuo codice, `/clear` per ripulire la conversazione quando cambi argomento.

### 5.4 Esempio di file CLAUDE.md per il nostro progetto

Crea questo file nella radice del progetto dopo averlo inizializzato:

```markdown
# GymManager — Contesto Progetto

## Obiettivo
Applicazione desktop per gestione clienti e schede palestra.

## Stack
- Tauri 2 (shell desktop)
- Rust (backend, comandi Tauri)
- Svelte 5 + TypeScript (frontend)
- SQLite via sqlx (database)
- TailwindCSS (styling)

## Convenzioni
- Codice in inglese, commenti in italiano dove utili.
- Errori Rust modellati con `thiserror`, restituiti al frontend come stringhe serializzate.
- Componenti Svelte in PascalCase, file in kebab-case.
- Migrazioni SQL versionate in `src-tauri/migrations/`.

## Struttura
- `src/` — frontend Svelte
- `src-tauri/src/` — backend Rust
- `src-tauri/migrations/` — SQL migrations
- `docs/` — documentazione tecnica

## Priorità
1. Affidabilità dei dati (no perdita di dati cliente).
2. Esperienza utente fluida.
3. Codice manutenibile e testato.
```

---

## 6. Creazione del Progetto Tauri + Svelte

### 6.1 Inizializzazione

Apri il terminale nella cartella dove vuoi creare il progetto e lancia:

```bash
pnpm create tauri-app
```

Rispondi alle domande:

- **Project name:** `gym-manager`
- **Identifier:** `com.tuonome.gymmanager` (formato reverse-domain)
- **Frontend language:** TypeScript / JavaScript
- **Package manager:** pnpm
- **UI template:** Svelte
- **UI flavor:** TypeScript

### 6.2 Prima esecuzione

```bash
cd gym-manager
pnpm install
pnpm tauri dev
```

La prima compilazione richiederà parecchi minuti (Rust deve compilare tutte le dipendenze). Le successive saranno molto più veloci. Si aprirà una finestra desktop con la pagina di benvenuto di Tauri.

### 6.3 Struttura del progetto

```
gym-manager/
├── src/                    # Frontend Svelte
│   ├── lib/                # Componenti riutilizzabili
│   ├── routes/             # Pagine (se usi SvelteKit) o viste
│   ├── App.svelte          # Componente root
│   └── main.ts             # Entry point JS
├── src-tauri/              # Backend Rust
│   ├── src/
│   │   ├── main.rs         # Entry point Rust
│   │   └── lib.rs          # Logica principale
│   ├── Cargo.toml          # Dipendenze Rust
│   ├── tauri.conf.json     # Configurazione Tauri
│   └── icons/              # Icone dell'app
├── package.json            # Dipendenze JS
└── README.md
```

### 6.4 Installazione di TailwindCSS

Per uno styling rapido e professionale, aggiungi Tailwind:

```bash
pnpm add -D tailwindcss postcss autoprefixer
pnpm dlx tailwindcss init -p
```

Configura `tailwind.config.js`:

```javascript
export default {
  content: ['./index.html', './src/**/*.{svelte,js,ts}'],
  theme: { extend: {} },
  plugins: [],
}
```

In `src/app.css` (creando il file se non esiste) aggiungi:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

E importalo in `src/main.ts`:

```typescript
import './app.css'
```

---

## 7. Configurazione di Git e GitHub

Versionare il codice **dal primo giorno** è non negoziabile per un professionista.

### 7.1 Inizializza il repository locale

Dalla cartella del progetto:

```bash
git init
```

Tauri ha già creato un `.gitignore` adeguato. Verifica che escluda almeno `node_modules/`, `target/`, `dist/`, `*.db`.

### 7.2 Primo commit

```bash
git add .
git commit -m "chore: initial project scaffold with tauri + svelte"
```

### 7.3 Convenzione dei commit

Usa **Conventional Commits** (https://www.conventionalcommits.org). Formato:

```
<tipo>(<scope opzionale>): <descrizione breve>
```

Tipi comuni: `feat`, `fix`, `chore`, `docs`, `refactor`, `test`, `style`, `perf`. Esempi:

- `feat(clients): add client creation form`
- `fix(db): handle duplicate email error`
- `docs: update setup instructions`

Questa convenzione rende lo storico leggibile, abilita generazione automatica del changelog e segnala l'intento delle modifiche.

### 7.4 Crea il repository su GitHub

Vai su https://github.com, crea un nuovo repository chiamato `gym-manager` (puoi tenerlo privato). **Non** inizializzarlo con README, license o gitignore (li hai già).

Collega il repo locale a quello remoto:

```bash
git remote add origin git@github.com:tuonome/gym-manager.git
git branch -M main
git push -u origin main
```

Se non hai configurato chiavi SSH, usa l'URL HTTPS e GitHub ti chiederà un Personal Access Token al posto della password.

### 7.5 Workflow di branching

Per ogni nuova funzionalità o fix, crea un branch dedicato:

```bash
git checkout -b feat/client-management
# ... lavora, fai commit ...
git push -u origin feat/client-management
```

Su GitHub apri una **Pull Request** verso `main`. Anche se lavori da solo, questo workflow ti abitua alla disciplina professionale: le PR sono il punto in cui rivedi il tuo codice prima di integrarlo.

### 7.6 GitHub Actions (CI)

Crea `.github/workflows/ci.yml` per eseguire automaticamente build e test su ogni push:

```yaml
name: CI
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - uses: dtolnay/rust-toolchain@stable
      - name: Install system dependencies
        run: |
          sudo apt-get update
          sudo apt-get install -y libwebkit2gtk-4.1-dev libxdo-dev libssl-dev libayatana-appindicator3-dev librsvg2-dev
      - name: Install pnpm
        run: npm install -g pnpm
      - run: pnpm install
      - run: pnpm check
      - name: Rust tests
        run: cargo test --manifest-path src-tauri/Cargo.toml
```

---

## 8. Architettura dell'Applicazione

### 8.1 Modello generale

```
┌──────────────────────────────────────────────────┐
│              FRONTEND (Svelte)                   │
│  Componenti UI, gestione stato, routing          │
└────────────────────┬─────────────────────────────┘
                     │ invoke() — chiamate ai comandi
                     ▼
┌──────────────────────────────────────────────────┐
│              TAURI BRIDGE                        │
│  Serializzazione, sicurezza, IPC                 │
└────────────────────┬─────────────────────────────┘
                     │
                     ▼
┌──────────────────────────────────────────────────┐
│              BACKEND (Rust)                      │
│  Logica di business, comandi Tauri               │
└────────────────────┬─────────────────────────────┘
                     │
                     ▼
┌──────────────────────────────────────────────────┐
│              SQLite (via sqlx)                   │
│  File .db locale, transazioni, query             │
└──────────────────────────────────────────────────┘
```

### 8.2 Principio di separazione

Il **frontend** non parla mai direttamente con il database. Tutta la persistenza passa attraverso **comandi Rust** esposti tramite `#[tauri::command]`. Questo separa nettamente le responsabilità: l'UI si occupa di presentazione, Rust di logica e dati.

### 8.3 Struttura interna del backend Rust

```
src-tauri/src/
├── main.rs              # Entry point
├── lib.rs               # Registrazione comandi e setup
├── db/                  # Layer database
│   ├── mod.rs
│   ├── pool.rs          # Connection pool
│   └── migrations.rs    # Esecuzione migrations
├── models/              # Strutture dati (Client, Workout, ...)
│   ├── mod.rs
│   ├── client.rs
│   ├── workout.rs
│   └── exercise.rs
├── commands/            # Comandi Tauri esposti al frontend
│   ├── mod.rs
│   ├── clients.rs
│   └── workouts.rs
├── repositories/        # Query SQL incapsulate
│   ├── mod.rs
│   └── client_repo.rs
└── error.rs             # Tipi di errore unificati
```

### 8.4 Struttura del frontend Svelte

```
src/
├── lib/
│   ├── components/      # Componenti riutilizzabili (Button, Modal, ...)
│   ├── stores/          # Stato globale (Svelte stores)
│   ├── api/             # Wrapper su invoke() per chiamare il backend
│   └── types.ts         # Tipi TypeScript condivisi
├── routes/              # Viste/pagine
│   ├── Dashboard.svelte
│   ├── Clients.svelte
│   ├── ClientDetail.svelte
│   └── Workouts.svelte
├── App.svelte
└── main.ts
```

---

## 9. Progettazione del Database SQLite

### 9.1 Schema iniziale

Le tabelle principali per il nostro dominio. Crea il file `src-tauri/migrations/0001_initial.sql`:

```sql
-- Clienti
CREATE TABLE clients (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    first_name TEXT NOT NULL,
    last_name TEXT NOT NULL,
    email TEXT UNIQUE,
    phone TEXT,
    birth_date DATE,
    gender TEXT CHECK(gender IN ('M', 'F', 'Other')),
    notes TEXT,
    photo_path TEXT,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Abbonamenti
CREATE TABLE memberships (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    client_id INTEGER NOT NULL REFERENCES clients(id) ON DELETE CASCADE,
    plan_name TEXT NOT NULL,
    start_date DATE NOT NULL,
    end_date DATE NOT NULL,
    price_cents INTEGER NOT NULL,
    paid INTEGER NOT NULL DEFAULT 0,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Esercizi (libreria)
CREATE TABLE exercises (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL UNIQUE,
    muscle_group TEXT NOT NULL,
    description TEXT,
    video_url TEXT,
    image_path TEXT
);

-- Schede di allenamento
CREATE TABLE workouts (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    client_id INTEGER NOT NULL REFERENCES clients(id) ON DELETE CASCADE,
    title TEXT NOT NULL,
    goal TEXT,
    start_date DATE NOT NULL,
    end_date DATE,
    notes TEXT,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Esercizi all'interno di una scheda
CREATE TABLE workout_exercises (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    workout_id INTEGER NOT NULL REFERENCES workouts(id) ON DELETE CASCADE,
    exercise_id INTEGER NOT NULL REFERENCES exercises(id),
    day_number INTEGER NOT NULL,         -- giorno A/B/C => 1,2,3
    order_in_day INTEGER NOT NULL,
    sets INTEGER NOT NULL,
    reps TEXT NOT NULL,                  -- "8-10" o "12"
    rest_seconds INTEGER,
    weight_kg REAL,
    notes TEXT
);

-- Misurazioni periodiche
CREATE TABLE measurements (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    client_id INTEGER NOT NULL REFERENCES clients(id) ON DELETE CASCADE,
    measured_at DATE NOT NULL,
    weight_kg REAL,
    height_cm REAL,
    body_fat_percent REAL,
    chest_cm REAL,
    waist_cm REAL,
    hips_cm REAL,
    arm_cm REAL,
    thigh_cm REAL,
    notes TEXT
);

-- Indici per query frequenti
CREATE INDEX idx_workouts_client ON workouts(client_id);
CREATE INDEX idx_memberships_client ON memberships(client_id);
CREATE INDEX idx_measurements_client ON measurements(client_id);
```

### 9.2 Perché modellare così

Le **foreign key** con `ON DELETE CASCADE` garantiscono integrità: se elimini un cliente, spariscono i suoi abbonamenti, schede e misurazioni. Le **constraint** (`CHECK`, `UNIQUE`, `NOT NULL`) impediscono dati inconsistenti. Conservare i prezzi in centesimi (`price_cents` come `INTEGER`) evita errori di arrotondamento dei floating point.

---

## 10. Sviluppo del Backend Rust

### 10.1 Aggiungi dipendenze a `src-tauri/Cargo.toml`

```toml
[dependencies]
tauri = { version = "2", features = [] }
tauri-plugin-shell = "2"
serde = { version = "1", features = ["derive"] }
serde_json = "1"
sqlx = { version = "0.8", features = ["runtime-tokio", "sqlite", "chrono", "migrate"] }
tokio = { version = "1", features = ["full"] }
chrono = { version = "0.4", features = ["serde"] }
thiserror = "1"
anyhow = "1"
tracing = "0.1"
tracing-subscriber = "0.3"
```

### 10.2 Tipo di errore unificato (`src-tauri/src/error.rs`)

```rust
use serde::Serialize;
use thiserror::Error;

#[derive(Debug, Error)]
pub enum AppError {
    #[error("database error: {0}")]
    Database(#[from] sqlx::Error),
    #[error("not found")]
    NotFound,
    #[error("validation: {0}")]
    Validation(String),
    #[error("io error: {0}")]
    Io(#[from] std::io::Error),
}

// Tauri richiede che gli errori siano serializzabili
impl Serialize for AppError {
    fn serialize<S: serde::Serializer>(&self, s: S) -> Result<S::Ok, S::Error> {
        s.serialize_str(&self.to_string())
    }
}

pub type AppResult<T> = Result<T, AppError>;
```

### 10.3 Pool di connessione database (`src-tauri/src/db/pool.rs`)

```rust
use sqlx::sqlite::{SqlitePool, SqlitePoolOptions, SqliteConnectOptions};
use std::path::Path;
use std::str::FromStr;

pub async fn create_pool(db_path: &Path) -> Result<SqlitePool, sqlx::Error> {
    let url = format!("sqlite://{}", db_path.display());
    let options = SqliteConnectOptions::from_str(&url)?
        .create_if_missing(true)
        .foreign_keys(true)
        .journal_mode(sqlx::sqlite::SqliteJournalMode::Wal);

    let pool = SqlitePoolOptions::new()
        .max_connections(5)
        .connect_with(options)
        .await?;

    sqlx::migrate!("./migrations").run(&pool).await?;
    Ok(pool)
}
```

L'attivazione del **journal WAL** migliora le performance e la concorrenza in lettura. L'opzione `foreign_keys(true)` attiva il rispetto delle foreign key (in SQLite è opt-in).

### 10.4 Modello di dato (`src-tauri/src/models/client.rs`)

```rust
use chrono::NaiveDate;
use serde::{Deserialize, Serialize};

#[derive(Debug, Serialize, Deserialize, sqlx::FromRow)]
pub struct Client {
    pub id: i64,
    pub first_name: String,
    pub last_name: String,
    pub email: Option<String>,
    pub phone: Option<String>,
    pub birth_date: Option<NaiveDate>,
    pub gender: Option<String>,
    pub notes: Option<String>,
    pub photo_path: Option<String>,
}

#[derive(Debug, Deserialize)]
pub struct NewClient {
    pub first_name: String,
    pub last_name: String,
    pub email: Option<String>,
    pub phone: Option<String>,
    pub birth_date: Option<NaiveDate>,
    pub gender: Option<String>,
    pub notes: Option<String>,
}
```

### 10.5 Repository (`src-tauri/src/repositories/client_repo.rs`)

```rust
use crate::error::{AppError, AppResult};
use crate::models::client::{Client, NewClient};
use sqlx::SqlitePool;

pub async fn list(pool: &SqlitePool) -> AppResult<Vec<Client>> {
    let rows = sqlx::query_as::<_, Client>(
        "SELECT id, first_name, last_name, email, phone, birth_date, gender, notes, photo_path
         FROM clients ORDER BY last_name, first_name"
    )
    .fetch_all(pool)
    .await?;
    Ok(rows)
}

pub async fn create(pool: &SqlitePool, input: NewClient) -> AppResult<i64> {
    if input.first_name.trim().is_empty() {
        return Err(AppError::Validation("first_name is required".into()));
    }
    let result = sqlx::query(
        "INSERT INTO clients (first_name, last_name, email, phone, birth_date, gender, notes)
         VALUES (?, ?, ?, ?, ?, ?, ?)"
    )
    .bind(&input.first_name)
    .bind(&input.last_name)
    .bind(&input.email)
    .bind(&input.phone)
    .bind(&input.birth_date)
    .bind(&input.gender)
    .bind(&input.notes)
    .execute(pool)
    .await?;
    Ok(result.last_insert_rowid())
}

pub async fn delete(pool: &SqlitePool, id: i64) -> AppResult<()> {
    let res = sqlx::query("DELETE FROM clients WHERE id = ?")
        .bind(id)
        .execute(pool)
        .await?;
    if res.rows_affected() == 0 {
        return Err(AppError::NotFound);
    }
    Ok(())
}
```

### 10.6 Comandi Tauri (`src-tauri/src/commands/clients.rs`)

```rust
use crate::error::AppResult;
use crate::models::client::{Client, NewClient};
use crate::repositories::client_repo;
use sqlx::SqlitePool;
use tauri::State;

#[tauri::command]
pub async fn list_clients(pool: State<'_, SqlitePool>) -> AppResult<Vec<Client>> {
    client_repo::list(&pool).await
}

#[tauri::command]
pub async fn create_client(
    pool: State<'_, SqlitePool>,
    input: NewClient,
) -> AppResult<i64> {
    client_repo::create(&pool, input).await
}

#[tauri::command]
pub async fn delete_client(
    pool: State<'_, SqlitePool>,
    id: i64,
) -> AppResult<()> {
    client_repo::delete(&pool, id).await
}
```

### 10.7 Setup principale (`src-tauri/src/lib.rs`)

```rust
mod commands;
mod db;
mod error;
mod models;
mod repositories;

use tauri::Manager;

#[cfg_attr(mobile, tauri::mobile_entry_point)]
pub fn run() {
    tauri::Builder::default()
        .plugin(tauri_plugin_shell::init())
        .setup(|app| {
            let app_dir = app.path().app_data_dir().expect("no app data dir");
            std::fs::create_dir_all(&app_dir).ok();
            let db_path = app_dir.join("gym-manager.db");

            let pool = tauri::async_runtime::block_on(async {
                db::pool::create_pool(&db_path).await.expect("db init failed")
            });

            app.manage(pool);
            Ok(())
        })
        .invoke_handler(tauri::generate_handler![
            commands::clients::list_clients,
            commands::clients::create_client,
            commands::clients::delete_client,
        ])
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

---

## 11. Sviluppo del Frontend Svelte

### 11.1 Wrapper API tipizzato (`src/lib/api/clients.ts`)

```typescript
import { invoke } from '@tauri-apps/api/core'

export interface Client {
  id: number
  first_name: string
  last_name: string
  email?: string
  phone?: string
  birth_date?: string
  gender?: 'M' | 'F' | 'Other'
  notes?: string
  photo_path?: string
}

export interface NewClient {
  first_name: string
  last_name: string
  email?: string
  phone?: string
  birth_date?: string
  gender?: 'M' | 'F' | 'Other'
  notes?: string
}

export const clientsApi = {
  list: () => invoke<Client[]>('list_clients'),
  create: (input: NewClient) => invoke<number>('create_client', { input }),
  delete: (id: number) => invoke<void>('delete_client', { id }),
}
```

### 11.2 Store dei clienti (`src/lib/stores/clients.ts`)

```typescript
import { writable } from 'svelte/store'
import { clientsApi, type Client, type NewClient } from '../api/clients'

function createClientsStore() {
  const { subscribe, set, update } = writable<Client[]>([])

  return {
    subscribe,
    async load() {
      const list = await clientsApi.list()
      set(list)
    },
    async add(input: NewClient) {
      await clientsApi.create(input)
      await this.load()
    },
    async remove(id: number) {
      await clientsApi.delete(id)
      update(list => list.filter(c => c.id !== id))
    },
  }
}

export const clients = createClientsStore()
```

### 11.3 Componente lista clienti (`src/routes/Clients.svelte`)

```svelte
<script lang="ts">
  import { onMount } from 'svelte'
  import { clients } from '../lib/stores/clients'

  let firstName = ''
  let lastName = ''
  let email = ''

  onMount(() => clients.load())

  async function handleSubmit() {
    if (!firstName || !lastName) return
    await clients.add({ first_name: firstName, last_name: lastName, email })
    firstName = lastName = email = ''
  }
</script>

<section class="p-6">
  <h1 class="text-2xl font-bold mb-4">Clienti</h1>

  <form on:submit|preventDefault={handleSubmit} class="flex gap-2 mb-6">
    <input bind:value={firstName} placeholder="Nome" class="border px-2 py-1 rounded" />
    <input bind:value={lastName} placeholder="Cognome" class="border px-2 py-1 rounded" />
    <input bind:value={email} placeholder="Email" type="email" class="border px-2 py-1 rounded" />
    <button class="bg-blue-600 text-white px-3 py-1 rounded">Aggiungi</button>
  </form>

  <ul class="divide-y">
    {#each $clients as client (client.id)}
      <li class="flex justify-between py-2">
        <span>{client.first_name} {client.last_name} — {client.email ?? '—'}</span>
        <button on:click={() => clients.remove(client.id)} class="text-red-600">Elimina</button>
      </li>
    {/each}
  </ul>
</section>
```

---

## 12. Funzionalità Principali da Implementare

Procedi per **incrementi**: una funzionalità per volta, ognuna su un branch dedicato. Suggerimento di ordine professionale:

1. **CRUD Clienti** (creazione, lettura, aggiornamento, eliminazione) — il fondamento.
2. **Dettaglio cliente** con foto profilo e note estese.
3. **Misurazioni** con grafici di andamento (libreria `Chart.js` o `Apache ECharts`).
4. **Libreria esercizi** con filtro per gruppo muscolare.
5. **Schede di allenamento**: editor drag-and-drop per ordinare esercizi.
6. **Abbonamenti** con dashboard scadenze (avvisi per quelli in scadenza nei prossimi 7 giorni).
7. **Esportazione PDF** delle schede usando una libreria Rust come `printpdf`.
8. **Backup automatico** del database in una cartella scelta dall'utente.
9. **Ricerca globale** con `LIKE` o full-text search di SQLite (`FTS5`).
10. **Multi-utente / Login** se serve più operatori.

Per ogni feature segui questo ciclo:

1. Scrivi una breve **specifica** in `docs/features/NOME.md`.
2. Crea un **branch** e apri una **issue** GitHub.
3. Implementa **backend** (modello, repository, comandi, test).
4. Implementa **frontend** (componenti, store, integrazione).
5. **Testa manualmente** in `pnpm tauri dev`.
6. Apri **Pull Request**, leggi il diff, fai merge.

---

## 13. Testing e Qualità del Codice

### 13.1 Test Rust

Crea `src-tauri/tests/` per test di integrazione, o usa `#[cfg(test)] mod tests { ... }` nei file. Esempio:

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use sqlx::SqlitePool;

    async fn setup() -> SqlitePool {
        let pool = SqlitePool::connect(":memory:").await.unwrap();
        sqlx::migrate!("./migrations").run(&pool).await.unwrap();
        pool
    }

    #[tokio::test]
    async fn creates_and_lists_client() {
        let pool = setup().await;
        let id = client_repo::create(&pool, NewClient {
            first_name: "Mario".into(),
            last_name: "Rossi".into(),
            ..Default::default()
        }).await.unwrap();
        assert!(id > 0);
        let list = client_repo::list(&pool).await.unwrap();
        assert_eq!(list.len(), 1);
    }
}
```

Esegui con `cargo test`.

### 13.2 Test frontend

Aggiungi **Vitest** per test unitari Svelte:

```bash
pnpm add -D vitest @testing-library/svelte jsdom
```

### 13.3 Linting e formattazione

Rust: `cargo fmt` e `cargo clippy -- -D warnings` (clippy è il linter di Rust).

Frontend: aggiungi **Prettier** + **ESLint**:

```bash
pnpm add -D prettier prettier-plugin-svelte eslint
```

Aggancia tutto a **pre-commit hooks** con `husky` + `lint-staged` per eseguire il check automaticamente prima di ogni commit. Niente codice non formattato finisce mai nel repo.

### 13.4 Code review con Claude Code

Prima di aprire una PR, in Claude Code:

```
/review
```

Claude esaminerà i tuoi diff cercando bug, anti-pattern, problemi di sicurezza e suggerimenti di miglioramento.

---

## 14. Build, Packaging e Distribuzione

### 14.1 Build di sviluppo

```bash
pnpm tauri dev
```

### 14.2 Build di produzione

```bash
pnpm tauri build
```

Genera in `src-tauri/target/release/bundle/`:

- **Windows:** `.msi` e `.exe` (NSIS installer)
- **macOS:** `.dmg` e `.app`
- **Linux:** `.deb`, `.rpm`, `.AppImage`

### 14.3 Icone

Sostituisci le icone di default in `src-tauri/icons/`. Tauri include un comando per generarle tutte da un singolo PNG ad alta risoluzione:

```bash
pnpm tauri icon path/to/source-icon.png
```

### 14.4 Versionamento

Aggiorna `version` in `src-tauri/tauri.conf.json` e in `package.json` seguendo **Semantic Versioning** (`MAJOR.MINOR.PATCH`). Aggiungi un **CHANGELOG.md** che documenti le modifiche di ogni versione.

### 14.5 Release automatica con GitHub Actions

Configura una workflow che, al push di un tag `v*`, builda l'app per Windows, macOS e Linux e crea una **GitHub Release** con i binari allegati. Tauri fornisce un'action ufficiale (`tauri-apps/tauri-action`) che fa tutto questo automaticamente.

### 14.6 Firma del codice

Per distribuire un'app senza far apparire avvisi di sicurezza agli utenti, dovrai **firmare** i binari:

- **Windows:** certificato di code signing (a pagamento da CA come Sectigo o DigiCert).
- **macOS:** Apple Developer Program ($99/anno) per firmare e notarizzare.

Per uso interno o test puoi saltare questo passaggio.

---

## 15. Best Practice Professionali

### 15.1 Documentazione

Mantieni aggiornati:

- **README.md** con descrizione, requisiti, setup, comandi principali.
- **CONTRIBUTING.md** con linee guida per contributi.
- **docs/** con architettura, decisioni tecniche (ADR — Architecture Decision Records) e tutorial.
- Commenti nel codice solo dove il **perché** non è ovvio dal **cosa**.

### 15.2 Sicurezza

Non committare mai segreti (password, token, chiavi). Usa `.env` con `.env.example` versionato. Su Tauri configura attentamente la **allowlist** in `tauri.conf.json`: concedi solo i permessi che servono davvero. Valida sempre l'input lato Rust, non fidarti del frontend.

### 15.3 Gestione dei dati utente

I dati dei clienti sono potenzialmente sotto GDPR. Implementa:

- Esportazione dei dati di un cliente in un formato leggibile (JSON o PDF).
- Eliminazione completa (right to be forgotten).
- Backup cifrato opzionale.
- Log delle operazioni critiche.

### 15.4 Performance

SQLite è veloce, ma sui dataset grandi servono indici sui campi di ricerca. Per migliaia di clienti niente paura: SQLite gestisce milioni di record senza problemi. Usa **paginazione** nelle liste lunghe.

### 15.5 Accessibilità

Usa elementi HTML semantici (`<button>`, `<label>`, `<nav>`), label associate ai campi, gestione tastiera, contrasto adeguato. È buona pratica anche se usano l'app solo poche persone.

### 15.6 Internazionalizzazione

Anche se inizi solo in italiano, usa una libreria di i18n (`svelte-i18n`) fin dall'inizio: estrarre i testi più tardi è doloroso.

### 15.7 Log e telemetria

Usa `tracing` lato Rust per log strutturati. In sviluppo stampali a console, in produzione scrivili in un file `logs/` rotante. Non mandare mai dati personali in log che potrebbero finire fuori dalla macchina dell'utente.

---

## 16. Risoluzione Problemi Comuni

**`pnpm tauri dev` non si avvia / errore WebView2 su Windows.** Installa manualmente WebView2 Runtime da Microsoft.

**Errori di compilazione `libwebkit2gtk` su Linux.** Su distribuzioni recenti potresti dover installare la versione `4.1`. Aggiorna i pacchetti come indicato in 4.5.

**Database "locked".** Stai aprendo il file con un altro programma (es. SQLite browser) durante l'esecuzione dell'app. Chiudilo.

**Le migrazioni non vengono applicate.** Controlla che la cartella `src-tauri/migrations/` esista al momento della compilazione e che i nomi seguano il formato `NNNN_descrizione.sql`.

**Compilazione lentissima.** Normale la prima volta (15-30 min). Successive build incrementali < 30 secondi. Aggiungi `[profile.dev] opt-level = 1` in `Cargo.toml` per velocizzare l'esecuzione in dev.

**Tipi TypeScript desincronizzati con Rust.** Considera l'uso del crate `specta` o `ts-rs` che generano automaticamente i tipi TS dalle struct Rust.

---

## 17. Roadmap di Apprendimento

Mentre costruisci l'app, dedica tempo regolare allo studio dei singoli pezzi. Una roadmap consigliata:

**Settimana 1-2 — Fondamenta.** Completa "The Rust Book" (https://doc.rust-lang.org/book/) almeno fino al capitolo 10. Guarda il tutorial ufficiale Svelte (https://learn.svelte.dev).

**Settimana 3-4 — Setup completo.** Segui questa guida fino al capitolo 11, hai CRUD Clienti funzionante.

**Settimana 5-8 — Funzionalità.** Implementa una feature dalla sezione 12 per settimana. Ogni feature è un branch, una PR, una review.

**Settimana 9-10 — Testing e qualità.** Aggiungi test ai punti chiave, configura CI, riconcilia il debito tecnico.

**Settimana 11-12 — Distribuzione.** Build di produzione, icone, installer, prima release v1.0.0.

Dopo: itera. Raccogli feedback da utenti reali (anche solo un amico personal trainer), correggi bug, aggiungi funzionalità. È così che si diventa professionisti — costruendo, sbagliando e migliorando.

---

## Conclusione

Hai in mano una mappa completa. Non leggere tutto e poi iniziare: alterna lettura e pratica. Apri il terminale, segui il capitolo 4, fai tutto. Poi torna qui per il capitolo successivo.

Tre principi finali:

**Commit spesso, push spesso.** Il codice non versionato non esiste. Un commit ogni 30-60 minuti di lavoro è un buon ritmo.

**Chiedi a Claude Code, ma capisci la risposta.** Se accetti codice che non capisci, stai accumulando debito di conoscenza. Fermati e chiedi spiegazioni.

**Spedisci.** Un'app v1 imperfetta che gira su un computer vero vale infinitamente di più di un'app perfetta che esiste solo nella tua testa.

Buon coding.
