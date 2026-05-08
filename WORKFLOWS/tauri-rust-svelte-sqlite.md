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
