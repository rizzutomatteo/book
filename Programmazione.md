# Workflow Professionale per Sviluppatore Singolo

Linea guida operativa completa per progetti software gestiti in autonomia con standard professionali. Copre l'intero ciclo di vita: dal setup iniziale del repository al deploy in produzione, dalla manutenzione continua al disaster recovery. Basata sul plugin [everything-claude-code](https://github.com/affaan-m/everything-claude-code) per la parte di sviluppo assistito, e su pratiche industry-standard per il resto.

Il documento è strutturato in cinque parti:

- **Parte I** — Setup iniziale (one-time per progetto)
- **Parte II** — Ciclo di sviluppo (ricorrente per feature)
- **Parte III** — Infrastruttura e Operations
- **Parte IV** — Manutenzione continua
- **Parte V** — Riferimenti

Nessun passaggio è implicito. Ogni fase ha un "cosa fare se fallisce".

---

## Principi Generali (rileggere ogni inizio progetto)

1. **Niente codice prima di Parte I + Fase 0.** Non si apre l'editor finché setup repository e scope non sono completi.
2. **Slash command quando esistono, prosa solo per contesto extra.** Le slash command sono deterministiche e affidabili nel tempo.
3. **`/compact` tra le fasi maggiori.** Tiene il contesto pulito.
4. **`/checkpoint` ogni volta che termini una sotto-attività che funziona.**
5. **TDD adattivo, non dogmatico.** Coverage variabile per criticità (vedi Fase 4).
6. **Security review obbligatoria** se il progetto ha auth, input utente, database, o endpoint pubblici.
7. **Documenta mentre lavori, non dopo.**
8. **Nessun secret in repository, mai.** Senza eccezioni.
9. **Backup automatici e testati.** Un backup mai testato non è un backup.
10. **Timebox tutto quello che rischia di diventare un rabbit-hole.**

---

# PARTE I — Setup Iniziale del Progetto

Eseguito una volta per progetto, prima di scrivere codice. Tempo stimato: 2-4 ore. È un investimento che si ripaga decine di volte.

## A.1 — Repository GitHub

### A.1.1 — Creazione

1. Crea il repo su GitHub. **Privato di default**, anche per progetti che diventeranno open source: meglio aprire dopo, che chiudere dopo.
2. Inizializzalo **vuoto** (senza README, .gitignore, license generati da GitHub) — li gestirai tu localmente, così sono sotto controllo dal primo commit.
3. Clona localmente:
   ```bash
   git clone git@github.com:<utente>/<repo>.git
   cd <repo>
   ```

### A.1.2 — File obbligatori alla root

Crea subito questi file, anche minimali:

- `README.md` — titolo, descrizione una riga, sezione "Quick start" (anche solo placeholder), link a `docs/`.
- `LICENSE` — vedi A.8.
- `.gitignore` — vedi A.1.3.
- `.editorconfig` — uniforma indentazione e line ending fra editor diversi.
- `CHANGELOG.md` — formato [Keep a Changelog](https://keepachangelog.com/), inizia con sezione `[Unreleased]`.

### A.1.3 — `.gitignore` esaustivo

Genera la base da [gitignore.io](https://www.toptal.com/developers/gitignore) per il tuo stack, poi aggiungi sempre:

```
# Secrets
.env
.env.local
.env.*.local
*.pem
*.key
secrets/

# IDE
.idea/
.vscode/
*.swp

# OS
.DS_Store
Thumbs.db

# Build artifacts
dist/
build/
*.log

# Coverage
coverage/
.nyc_output/
```

Verifica con `git check-ignore -v <file>` che i file critici siano ignorati prima del primo commit.

### A.1.4 — Branch protection rules

Su GitHub: Settings → Branches → Add rule per `main`:

- **Require a pull request before merging** (anche solo per te: forza il workflow).
- **Require status checks to pass before merging** → spunta i job CI che configurerai in A.7.
- **Require branches to be up to date before merging**.
- **Do not allow bypassing the above settings** (sì, anche per te owner — protegge da errori distratti).
- **Require linear history** (impone squash o rebase, niente merge commits sporchi).

### A.1.5 — Repository config

Settings → General:

- **Default branch**: `main`.
- **Features**: disabilita Wiki e Projects se usi alternative; tieni Issues attivo.
- **Pull Requests**: attiva "Allow squash merging", disattiva "Allow merge commits" (storia lineare). Attiva "Automatically delete head branches".

### A.1.6 — Templates `.github/`

Crea la cartella `.github/` con:

- `pull_request_template.md`:
  ```markdown
  ## Cosa cambia
  
  ## Perché
  
  ## Come testare
  
  ## Checklist
  - [ ] Test aggiunti/aggiornati
  - [ ] Documentazione aggiornata
  - [ ] Coverage rispettata (vedi tabella in workflow)
  - [ ] /verification-loop PASS
  - [ ] Security review se applicabile
  - [ ] CHANGELOG aggiornato
  ```
- `ISSUE_TEMPLATE/bug_report.md` e `ISSUE_TEMPLATE/feature_request.md` — template minimi per non rendere caotico il backlog.
- `CODEOWNERS` con `* @<tuo-username>` — banale ma utile come documentazione e nel caso aprissi il progetto a contributori.

### A.1.7 — Issue tracking / Kanban personale

Anche da solo, usa un sistema di tracking. Opzioni:

- **GitHub Projects** (Settings → Projects, attiva e crea una board collegata al repo). Vista Kanban con colonne `Backlog`, `Todo`, `In progress`, `Review`, `Done`.
- **Linear/Jira/Notion** se preferisci tool dedicati.

Regola: **ogni feature parte da una issue**. Niente "lavoro a memoria" su feature non tracciate. La issue diventa la storia di cosa hai fatto.

## A.2 — License

Decidi la licenza prima del primo commit, non dopo. Linee guida:

- **MIT** — progetti open dove vuoi adozione massima e non ti importano i derivati chiusi.
- **Apache 2.0** — come MIT ma con clausola brevetti esplicita. Default sicuro per OSS aziendale.
- **GPL v3 / AGPL v3** — vuoi forzare i derivati a restare open. AGPL chiude il "loophole" del SaaS.
- **Proprietary / All rights reserved** — codice chiuso. File LICENSE con `Copyright (c) YYYY <Nome>. All rights reserved.`
- **Dual licensing** — open + commerciale, per progetti che vuoi monetizzare.

In dubbio: [choosealicense.com](https://choosealicense.com/). Documenta la scelta in `docs/adr/0001-licenza.md`.

## A.3 — Local Development Environment

### A.3.1 — Version manager

Usa un version manager per il runtime principale, mai la versione di sistema:

- Node: `nvm`, `fnm`, o `volta`.
- Python: `pyenv` + `pipx`, oppure `uv`.
- Ruby: `rbenv` o `asdf`.
- Multi-runtime: `asdf` o `mise`.

File `.nvmrc` / `.python-version` / `.tool-versions` committato, così la versione è esplicita.

### A.3.2 — Task runner

Aggiungi un `Makefile` (o `justfile`, `Taskfile.yml`) alla root con i comandi più frequenti:

```makefile
.PHONY: install dev test lint typecheck build clean docker-up docker-down

install:
	npm ci

dev:
	npm run dev

test:
	npm test -- --coverage

lint:
	npm run lint

typecheck:
	npm run typecheck

build:
	npm run build

clean:
	rm -rf dist node_modules

docker-up:
	docker compose up -d

docker-down:
	docker compose down
```

Documenta i target nel README. **Mai memorizzare comandi a mente**: scriveli qui.

### A.3.3 — Devcontainer (opzionale ma consigliato)

Per progetti complessi o multi-tool, configura un devcontainer (`.devcontainer/devcontainer.json`). Ti garantisce ambiente identico fra macchine e fra te-oggi e te-fra-sei-mesi. Essenziale se mai prendessi una macchina nuova o lavorassi temporaneamente da un'altra postazione.

## A.4 — Docker Setup

Anche per progetti piccoli, Docker è utile per: ambiente di dev riproducibile, deploy uniforme, isolamento delle dipendenze (DB, Redis, ecc.).

### A.4.1 — `Dockerfile` multi-stage

Pattern standard:

```dockerfile
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
```

Principi non negoziabili:

- **Multi-stage** sempre, per non includere dev dependencies e tool di build nell'immagine finale.
- **Utente non-root** sempre. Mai `USER root` in runtime.
- **Base image minimale**: `alpine`, `distroless`, `slim`. Mai `latest`, sempre tag versionato.
- **HEALTHCHECK** sempre.
- **`.dockerignore`** sempre (vedi A.4.2).
- **Cache mount** per package manager (`--mount=type=cache`) per build veloci.
- **Niente secret in immagine.** Mai `ENV API_KEY=...`. I secret entrano a runtime via env o secret manager.

### A.4.2 — `.dockerignore`

```
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
```

### A.4.3 — `docker-compose.yml` per dev

Per orchestrare app + servizi (DB, Redis, ecc.) in locale:

```yaml
services:
  app:
    build:
      context: .
      target: build  # stage non-runtime per development con hot reload
    ports:
      - "3000:3000"
    volumes:
      - .:/app
      - /app/node_modules
    env_file: .env.local
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: dev
      POSTGRES_PASSWORD: dev
      POSTGRES_DB: app_dev
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
```

`docker-compose.prod.yml` separato per production (override).

### A.4.4 — Image tagging

Strategia di tag obbligatoria:

- `<repo>:latest` — solo per dev, **mai per deploy production**.
- `<repo>:<git-sha>` — ogni build, immutabile, tracciabile.
- `<repo>:<semver>` (es. `v1.2.3`) — per release.
- `<repo>:<branch>` — opzionale, per preview environments.

Deploy in produzione **solo da tag semver o sha**, mai da `latest`.

### A.4.5 — Image scanning

Scansione vulnerabilità a ogni build, in CI:

```bash
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy image --severity HIGH,CRITICAL --exit-code 1 <image>:<tag>
```

Trivy (gratis, open source) è lo standard di fatto. Failure su HIGH/CRITICAL blocca il deploy.

### A.4.6 — Registry

Scegli uno e committi:

- **GitHub Container Registry (GHCR)** — gratis per repo pubblici, integrato con GitHub Actions, gestisce permessi via repo. **Default consigliato** per progetti su GitHub.
- **Docker Hub** — universale ma rate limit aggressivi sul piano free.
- **Cloud-native** (ECR, GCR, Artifact Registry) — se deployate sullo stesso cloud.

## A.5 — Secret Management

### A.5.1 — Regole non negoziabili

- **Mai** secret committati. Né in chiaro, né "temporaneamente", né in branch privati.
- **Mai** secret in `Dockerfile`, `docker-compose.yml` di esempio, o in CI workflow file in chiaro.
- **Mai** stesso secret in più ambienti. Dev, staging, production hanno credenziali distinte.
- **Rotazione** annuale almeno per secret long-lived. Subito dopo qualsiasi sospetto leak.

### A.5.2 — File pattern

- `.env` — locale, **ignorato da Git** (vedi A.1.3).
- `.env.example` — committato, contiene **chiavi senza valori**, documenta cosa serve:
  ```
  DATABASE_URL=
  JWT_SECRET=
  STRIPE_API_KEY=
  ```
- `.env.production` — **mai locale, mai committato**. Vive solo nel secret manager o nelle env del server.

### A.5.3 — Secret manager

Scegli uno:

- **1Password CLI (`op`)** — ottimo per uso personale, sync fra dispositivi, integrazione con team se serve.
- **Doppler** — gratis fino a un certo livello, gestione multi-environment, integrazione CI nativa.
- **Cloud-native** (AWS Secrets Manager, GCP Secret Manager, Azure Key Vault) — se sei già su un cloud.
- **HashiCorp Vault self-hosted** — overkill per solo, considera solo se hai esigenze regulatory.

Documenta in `docs/secrets.md`: dove vivono i secret, come ruotarli, chi li può accedere (anche solo "tu via 1Password").

### A.5.4 — Pre-commit secret scan

Configura un pre-commit hook (vedi A.6) che lancia [`gitleaks`](https://github.com/gitleaks/gitleaks) o `trufflehog`:

```bash
gitleaks protect --staged --redact
```

Se rileva un possibile secret, il commit è bloccato.

### A.5.5 — Cosa fare se hai committato un secret

1. **Considera il secret compromesso.** Anche se hai fatto force-push subito, supponi che qualcuno l'abbia visto.
2. **Ruota il secret immediatamente** sul provider.
3. **Riscrivi la storia**: `git filter-repo` (preferito) o BFG Repo-Cleaner per rimuovere il secret da tutta la storia.
4. **Force-push** dopo aver coordinato (se altri hanno cloni, anche tu su altre macchine).
5. **Scrivi un post mortem** in `docs/incidents/`.

## A.6 — Pre-commit Hooks

### A.6.1 — Tool

Scegli uno:

- **lefthook** — veloce, scritto in Go, config in `lefthook.yml`. **Default consigliato.**
- **husky + lint-staged** — standard JavaScript ecosystem.
- **pre-commit** (Python) — eccellente con vasto catalogo di hook pronti.

### A.6.2 — Cosa farci girare

Hook devono essere **veloci** (sotto 5-10 secondi), altrimenti li disabiliterai. Cosa includere:

- **Format check** (prettier --check, gofmt -l, black --check).
- **Lint sui file modificati** (eslint, ruff, golangci-lint).
- **Type check incrementale** (tsc --noEmit, mypy --incremental).
- **Secret scan** (gitleaks).
- **Commit message lint** (commitlint per Conventional Commits).

**Non** includere: test pesanti, build completi, integration test. Quelli vivono in CI (vedi A.7).

### A.6.3 — Esempio `lefthook.yml`

```yaml
pre-commit:
  parallel: true
  commands:
    lint:
      glob: "*.{js,ts,jsx,tsx}"
      run: npx eslint {staged_files}
    format:
      glob: "*.{js,ts,jsx,tsx,json,md}"
      run: npx prettier --check {staged_files}
    typecheck:
      glob: "*.{ts,tsx}"
      run: npx tsc --noEmit
    secrets:
      run: gitleaks protect --staged --redact

commit-msg:
  commands:
    commitlint:
      run: npx commitlint --edit {1}
```

### A.6.4 — Skip in emergenza

`git commit --no-verify` salta gli hook. Usalo **solo** per WIP commit su branch di lavoro, **mai** per commit su main o per PR. Documenta in `docs/conventions.md` quando è ammesso.

## A.7 — CI/CD Pipeline

### A.7.1 — Workflow minimo (GitHub Actions)

`.github/workflows/ci.yml`:

```yaml
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
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version-file: .nvmrc
          cache: npm
      - run: npm ci
      - run: npm run lint
      - run: npm run typecheck
      - run: npm test -- --coverage
      - uses: codecov/codecov-action@v4
        with:
          token: ${{ secrets.CODECOV_TOKEN }}

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
      - name: Trivy scan
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ghcr.io/${{ github.repository }}:sha-${{ github.sha }}
          severity: HIGH,CRITICAL
          exit-code: '1'

  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npx ecc-agentshield scan
```

### A.7.2 — Workflow di deploy

`.github/workflows/deploy.yml` separato, triggerato su tag semver:

```yaml
name: Deploy

on:
  push:
    tags: ['v*']
  workflow_dispatch:  # permette deploy manuale

jobs:
  deploy-staging:
    runs-on: ubuntu-latest
    environment: staging
    steps:
      # ... build e deploy su staging
      - run: ./scripts/smoke-test.sh

  deploy-production:
    runs-on: ubuntu-latest
    environment: production  # richiede approval manuale via GitHub Environments
    needs: deploy-staging
    steps:
      # ... deploy su production
      - run: ./scripts/health-check.sh
```

### A.7.3 — GitHub Environments

Su GitHub: Settings → Environments. Crea `staging` e `production`:

- **Production**: spunta "Required reviewers" e mettici te stesso. Anche da solo: forza una pausa di 5 secondi prima del deploy production, evita errori distratti.
- Aggiungi i secret per environment (DATABASE_URL prod separato da staging, ecc.).

### A.7.4 — Caching

Configura cache aggressiva:

- Dependencies (`actions/setup-node` con `cache: npm`).
- Build artifacts fra job (`actions/upload-artifact` + `actions/download-artifact`).
- Docker layers (`cache-from: type=gha`).

Differenza fra CI veloce e CI lento è fra "rieseguo dopo ogni piccola modifica" e "evito perché impiega 15 minuti".

### A.7.5 — Cosa fare se la CI fallisce

- **Test fail in CI ma passa in locale**: differenza ambiente. Allinea (versione runtime, env vars, timezone).
- **CI flaky**: identifica il test, **non rilanciare alla cieca**. Aggiungi a `docs/tech-debt.md` come "test instabile" e pianifica fix.
- **Deploy fail**: rollback automatico se configurato, altrimenti procedura manuale (vedi Parte III).

## A.8 — Dependency Management

### A.8.1 — Dependabot

Crea `.github/dependabot.yml`:

```yaml
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
```

Alternativa più potente: **Renovate** (richiede installazione GitHub App, più config ma più flessibile).

### A.8.2 — Strategia di review delle PR Dependabot

- **Patch updates**: merge dopo CI verde, settimanale.
- **Minor updates**: merge dopo CI verde, leggi changelog se libreria critica.
- **Major updates**: **mai merge automatico**. Crea ADR se cambia API, leggi migration guide, esegui smoke test esteso.

Pianifica un'ora a settimana (es. lunedì mattina) per processare le PR Dependabot. **Non lasciarle stagnare**: dopo un mese diventano un debito ingestibile.

## A.9 — Documentazione iniziale

Crea la struttura `docs/` con questi file vuoti o minimali:

```
docs/
├── adr/                    # Architecture Decision Records
│   └── 0001-licenza.md
├── incidents/              # Post-mortem
├── spikes/                 # Spike reports
├── deploy.md               # Procedura di deploy
├── rollback.md             # Procedura di rollback
├── secrets.md              # Dove vivono i secret e come ruotarli
├── observability.md        # Cosa monitoriamo e dove
├── tech-debt.md            # Debito tecnico tracciato
├── security-exceptions.md  # Falsi positivi documentati
├── runbook.md              # Cosa fare quando qualcosa va storto
└── conventions.md          # Convenzioni del progetto
```

Anche se molti sono placeholder, averli pronti riduce l'attrito quando ti servono davvero.

---

# PARTE II — Ciclo di Sviluppo

Eseguito per ogni feature o milestone. Le Fasi 0-10 sono il cuore operativo del workflow.

## Fase 0 — Scoping della feature

### 0.1 — Verifica ambiente

```
/harness-audit
```

Se hai instinct salvati da progetti simili:

```
/instinct-import
```

### 0.2 — Issue su GitHub

Apri una issue (Parte I, A.1.7) con:

- Descrizione della feature.
- Criteri di accettazione (bullet list, testabili).
- Stima (in ore, moltiplicata per 1.5).
- Etichette (`feat`, `bug`, `chore`, ecc.).
- Se è una feature complessa: link a un eventuale ADR.

### 0.3 — Branch

```bash
git checkout main
git pull
git checkout -b feat/<short-description>-#<issue-number>
```

Convenzione di naming branch (vedi anche Parte V, "Branching strategy"):

- `feat/...` — nuova feature.
- `fix/...` — bug fix.
- `chore/...` — manutenzione (deps, tooling).
- `refactor/...` — refactor senza cambio comportamento.
- `docs/...` — solo documentazione.

### 0.4 — ADR se necessario

Se la feature implica una decisione architetturale non banale, crea `docs/adr/00XX-<titolo>.md` (template in Parte V).

### Cosa fare se Fase 0 fallisce

- **Non riesci a definire criteri di accettazione testabili**: la feature è troppo vaga. Spike di 2 ore prima.
- **Stima > tempo disponibile**: dividi in più feature più piccole.

## Fase 1 — Search-First

### 1.1 — Comando

```
Devo costruire [descrizione feature].
Stack: [tecnologie scelte o da scegliere].
Vincoli: [vincoli tecnici/business rilevanti].
Usa search-first per trovare librerie, boilerplate o approcci esistenti
prima di scrivere qualsiasi cosa.
```

### 1.2 — Verifica licenza

Per ogni dipendenza candidata non ovvia:

```
Verifica la licenza di [nome libreria] e dimmi se è compatibile
con [licenza del tuo progetto, vedi A.2].
Segnala vincoli di attribution.
```

### 1.3 — Verifica supply chain

- Ultima release < 12 mesi.
- Maintainer attivi > 1 (o azienda affidabile dietro).
- Nessuna CVE HIGH/CRITICAL aperta.
- `npm audit` / `pip-audit` PASS.

### Cosa fare se Fase 1 fallisce

- **Trovi 3+ alternative equivalenti**: vai in Fase 2 con `/council`.
- **Niente esiste**: probabilmente buon segno (problema specifico del tuo dominio), ma rifletti se stai reinventando la ruota.

## Fase 2 — Decisione Architetturale (`/council`)

### 2.1 — Quando usarla

- 2+ opzioni in bilico dopo Fase 1.
- Decisione difficile da invertire.
- Errore costerebbe giorni.

**Non** usarla per scelte reversibili (logger, CSS framework, ecc.).

### 2.2 — Comando

```
Ho due opzioni: [A] vs [B].
Contesto: [progetto, vincoli, priorità, riferimento ADR].
Usa /council.
```

### 2.3 — Output

ADR in `docs/adr/00XX-<decisione>.md` con motivazione, voci di minoranza, e condizioni di rivalutazione futura.

### Cosa fare se Fase 2 fallisce

- **Verdetti contraddittori in run multipli**: ti mancano informazioni. Spike di 2 ore.
- **Tutte le opzioni sembrano cattive**: torna a Fase 0, lo scope è probabilmente sbagliato.

## Fase 3 — Pianificazione

### 3.1 — Piano dettagliato

```
Pianifica l'implementazione completa di [feature].
Stack: [tecnologie].
Criteri di accettazione: [da issue].
Produci: architettura, struttura file, task list ordinata, dipendenze, rischi noti.
```

Per progetti multi-fronte:

```
/multi-plan
```

### 3.2 — Salva in `docs/plan.md` o nella issue GitHub

Se è una piccola feature, aggiorna direttamente la issue con la task list. Se è grande, scrivi in `docs/plan.md` e linkala dalla issue.

### 3.3 — Compatta contesto prima di sviluppare

```
/compact Focus on implementing <feature>: <stack in breve>
```

### Cosa fare se Fase 3 fallisce

- **Piano > 30 task**: dividi in milestone, pianifica solo la prima.
- **Dipendenze cicliche**: problema di design, risolvi prima di iniziare.

## Fase 4 — Sviluppo (TDD adattivo)

### 4.1 — Coverage target per tipo di codice

| Tipo di codice | Coverage minima | Note |
|---|---|---|
| Auth, pagamenti, sicurezza, dati sensibili | 100% sui path critici | Test anche dei casi di errore |
| Logica di business core | 80% | Unit + integration |
| API/controller | 70% | Integration test sufficienti |
| Glue code, adapter, wiring | 50% o smoke test | Non over-testare |
| Spike/prototipi etichettati come tali | 0% | Da rifare se promossi a produzione |

### 4.2 — Comando per feature

```
Implementa [feature].
Segui il tdd-workflow:
- Prima i test (unit + integration)
- Poi l'implementazione
- Coverage target: [X%] (vedi tabella)
Stack: [tecnologie rilevanti].
Vincoli: [es. autenticazione, isolamento dati, rate limiting].
```

### 4.3 — Database migrations

Se la feature tocca lo schema:

- **Strumento**: Prisma Migrate, Knex, Flyway, Alembic, Liquibase. **Non** alterare schema a mano.
- **Reversibili sempre**: ogni migration ha un down. Test del rollback in dev.
- **Backward-compatible quando possibile**: aggiungi colonne nullable, deprecate progressivamente. Rende il deploy zero-downtime possibile.
- **Mai** modificare migration già committata. Solo nuove migration sopra.

### 4.4 — Disciplina anti rabbit-hole

Timebox mentale per feature. Se sfori del 50%:

```
/checkpoint
```

Poi rivaluta: stima sbagliata o over-engineering? Se over-engineering: rollback al checkpoint e taglia.

### 4.5 — Checkpoint frequenti

Ogni test verde nuovo, ogni sotto-feature funzionante:

```
/checkpoint
```

### 4.6 — Verifica copertura

```
/test-coverage
```

A fine feature, prima di passare alla successiva.

### 4.7 — Commit incrementali

Commit piccoli e frequenti **sul tuo branch** (non su main). Conventional commits (vedi Parte V):

```bash
git add <file>
git commit -m "feat(auth): add login endpoint"
```

### 4.8 — Quando saltare TDD

Solo per:

- Spike etichettati in `docs/spikes/`.
- Glue code verificabile a colpo d'occhio.
- Throwaway script.

### Cosa fare se Fase 4 fallisce

- **Test passano ma codice non funziona**: test sbagliati. Riscrivili.
- **Test bloccato > 30 min**: `/checkpoint` di rollback, spike di 1 ora isolato.
- **Feature > previsto**: dividi e ripianifica.

## Fase 5 — Code Review

### 5.1 — Review generica

```
/code-review
```

### 5.2 — Review specifica per linguaggio

```
/python-review
/go-review
/cpp-review
/rust-review
/kotlin-review
/flutter-review
```

### 5.3 — Refactor mirato (timebox 1h)

```
/refactor-clean
```

Se sfori, `/checkpoint`, decidi se rinviare e tracciare in `docs/tech-debt.md`.

### 5.4 — Anti-indulgenza

Le review automatiche tendono a essere generose. Forza un secondo passaggio:

```
Trova i 3 punti più deboli di questo codice. Non essere indulgente.
```

### Cosa fare se Fase 5 fallisce

- **Problemi gravi (architettura)**: rollback al checkpoint pre-feature, ripensa design, nuovo ADR.

## Fase 6 — Security Review

### 6.1 — Quando obbligatoria

Se il progetto ha **anche solo una** di queste:

- Autenticazione utente.
- Input da utenti.
- Query a database.
- Endpoint pubblici.
- Lettura/scrittura file con percorsi parametrizzati.
- Esecuzione di comandi shell o codice dinamico.
- Gestione di token, credenziali, secret.

Per progetti puramente locali single-user senza rete: documenta lo skip in `docs/scope.md`.

### 6.2 — Comando dedicato (default)

```
/security-review
```

### 6.3 — Aggiungi contesto se serve

```
Aggiungi a quanto già controllato anche: [contesto specifico,
es. "ho appena migrato l'auth da JWT a session cookie",
o "endpoint pubblico che accetta upload, controlla path traversal e MIME spoofing"].
```

### 6.4 — Scansione automatica

```bash
npx ecc-agentshield scan
```

1282 test. Risolvi tutti gli HIGH/CRITICAL. MEDIUM/LOW caso per caso, traccia in `docs/tech-debt.md` se rinviati.

### 6.5 — Container security scan

```bash
trivy image --severity HIGH,CRITICAL --exit-code 1 <repo>:<tag>
```

### 6.6 — Cosa fare se trovi un problema critico

1. **Stop**, niente Fase 7.
2. `/checkpoint`.
3. Fix focalizzato + test che dimostri la correzione.
4. Ripeti Fase 6 da zero.
5. Aggiorna ADR se la fix ha cambiato decisioni.

### Cosa fare se Fase 6 fallisce

- **Falsi positivi**: documenta in `docs/security-exceptions.md` con motivazione.
- **Problemi sistemici**: rollback al checkpoint pre-feature, ripensa design.

## Fase 7 — Verification Finale

### 7.1 — Verification loop completo

```
/verification-loop
```

Esegue: build, typecheck, lint, test con coverage, security scan. Stop al primo fail, report PASS/FAIL.

### 7.2 — Quality gate intermedio

```
/quality-gate
```

Differenza: `/quality-gate` è "sono a posto adesso?" durante lo sviluppo. `/verification-loop` è "pronto per PR?" a fine milestone.

### 7.3 — Cosa fare se fallisce

- **Build fail**: `/build-fix`.
- **Lint fail**: `--fix` automatico, poi residuo a mano.
- **Test fail**: torna a Fase 4 sulla feature colpevole.
- **Coverage sotto soglia**: aggiungi test, **mai abbassare la soglia**.
- **Security fail**: torna a Fase 6.

### 7.4 — Dopo 3 tentativi falliti

Stop. Problema strutturale. ADR o `docs/tech-debt.md` per documentare. Spike di 2 ore per causa radice.

## Fase 8 — Git, Pull Request, Merge

### 8.1 — Pulizia commit history

Sul tuo branch, prima del PR:

```bash
git fetch origin main
git rebase origin/main
```

Risolvi conflitti localmente. Se hai commit "WIP" o sporchi, fai squash:

```bash
git rebase -i origin/main
```

### 8.2 — Commit conventional

```
/prp-commit
```

Genera commit message con [Conventional Commits](https://www.conventionalcommits.org/):

```
feat(auth): add OAuth2 login flow

- Add /auth/oauth/callback endpoint
- Store refresh token encrypted
- Tests with coverage 95%

Closes #42
```

### 8.3 — Push e apri PR

```bash
git push origin feat/<branch>
```

Apri PR su GitHub. Usa il template (Parte I, A.1.6). Linka la issue (`Closes #42`).

### 8.4 — Genera descrizione PR

```
/prp-pr
```

### 8.5 — Self-review

```
/review-pr
```

Anche se sei tu, leggilo come se fossi un altro. Applica fix sensati, ignora nitpick con motivazione.

### 8.6 — Aspetta CI verde

CI deve passare tutti i job (test, build, docker, security). Se un job fallisce: torna a Fase 7.

### 8.7 — Merge

- **Squash and merge** (configurato in A.1.5): un solo commit pulito su main per PR.
- Verifica che il commit message squashato sia conventional.
- Branch eliminato automaticamente.

### 8.8 — Tag e release (se applicabile)

Se la PR chiude una milestone o bump di versione:

```bash
git checkout main
git pull
git tag -a v1.2.3 -m "Release v1.2.3"
git push origin v1.2.3
```

Su GitHub: Releases → Draft new release → seleziona il tag → genera release notes (GitHub lo fa automaticamente dai conventional commits).

### Cosa fare se Fase 8 fallisce

- **Conflitti merge**: risolvi sul branch con rebase, ri-esegui Fase 7.
- **CI verde locale ma rosso CI**: differenza ambiente. Investiga prima del merge.

## Fase 9 — Documentazione

### 9.1 — Update automatico

```
/update-docs
```

### 9.2 — Aggiorna a mano

- **README.md** se il setup o i comandi sono cambiati.
- **CHANGELOG.md** sotto `[Unreleased]` con la modifica.
- **ADR** se hai preso una decisione nuova.
- **API docs** (OpenAPI/JSDoc) se hai modificato API.
- **`docs/runbook.md`** se la feature richiede operatività diversa.

### 9.3 — Documentazione nello stesso PR

**Sempre**. Mai PR separato per docs: si dimentica.

## Fase 10 — Fine Sessione

### 10.1 — Comandi (in ordine)

```
/learn
/save-session
/evolve
```

### 10.2 — Push finale

```bash
git push
```

Se sei a fine giornata, push anche se la feature non è completata. Single point of failure mitigation.

### 10.3 — Backup mensile instinct

Una volta al mese:

```
/instinct-export
```

File esportato → repository separato versionato con Git (es. `~/dev/instinct-backups/`).

### 10.4 — Riprendere

A inizio sessione successiva:

```
/resume-session
```

---

# PARTE III — Infrastruttura e Operations

## P3.1 — Strategia di Deploy

### Ambienti

Minimo:

- **Local** — la tua macchina via Docker Compose.
- **Staging** — ambiente identico a production, dati anonimizzati o sintetici, deploy automatico da `main`.
- **Production** — ambiente reale, deploy manuale o da tag semver.

Nessuna eccezione: anche per progetti piccoli, almeno staging + production separate.

### Tipologie di deploy

Scegli in base al progetto:

- **Recreate** — stop vecchio, start nuovo. Downtime breve, semplice. OK per progetti hobby.
- **Rolling** — sostituzione progressiva delle istanze. Zero downtime se hai >1 istanza.
- **Blue-Green** — due ambienti identici, switch del traffico. Rollback istantaneo.
- **Canary** — deploy graduale al 5%, 25%, 50%, 100% del traffico. Per feature ad alto rischio.

Documenta la scelta in `docs/deploy.md` con comandi specifici.

### Pre-deploy checklist (esegui ogni volta)

1. Backup del database di produzione.
2. Tag di versione su Git (`v1.2.3`).
3. CI verde su tag.
4. `/verification-loop` PASS su staging.
5. Smoke test manuale dei flussi critici su staging.
6. Annota la versione precedente (rollback target).
7. Comunica il deploy (anche solo a te stesso, su un canale Slack personale o nel `docs/runbook.md`): "Deploy v1.2.3 a YYYY-MM-DD HH:MM".

### Procedura di deploy (`docs/deploy.md`)

Comandi copia-incolla, mai a memoria. Esempio:

```bash
# Deploy production v1.2.3
git checkout v1.2.3
docker build -t ghcr.io/user/app:v1.2.3 .
docker push ghcr.io/user/app:v1.2.3
ssh prod-server 'cd /app && docker compose pull && docker compose up -d'
./scripts/health-check.sh https://app.example.com
```

### Rollback (`docs/rollback.md`)

Anche qui, comandi copia-incolla:

```bash
# Rollback production a v1.2.2
ssh prod-server 'cd /app && \
  IMAGE=ghcr.io/user/app:v1.2.2 docker compose up -d'
./scripts/health-check.sh https://app.example.com
```

Tempo stimato di rollback: documentalo. Esegui un rollback di prova ogni 3 mesi su staging, per verificare che la procedura funzioni.

### Migration backward-incompatible

Se la migration non è reversibile (es. drop column con dati), il rollback del codice non basta:

1. Pianifica la migration in due step: prima codice nuovo che gestisce sia schema vecchio che nuovo, poi (release successiva) drop colonna vecchia.
2. **Mai** unire change di schema irreversibile e change di codice nello stesso deploy.
3. Backup pre-deploy obbligatorio.

## P3.2 — Observability

### Tre pilastri

1. **Logs** — cosa è successo.
2. **Metrics** — quanto/quanti.
3. **Traces** — dove e quanto è durato.

Solo il primo è obbligatorio per progetti piccoli. Aggiungi gli altri quando il debug solo da log diventa lento.

### Logs

- **Strutturati (JSON)** sempre, mai testo libero.
- **Livelli**: `error`, `warn`, `info`, `debug`. Mai `console.log` in produzione.
- **Campi obbligatori**: `timestamp` (ISO 8601 UTC), `level`, `message`, `service`, `request_id` se HTTP.
- **Mai** dati sensibili nei log: password, token, PII (email/telefono per la maggior parte dei casi). Ridazione automatica.
- **Centralizzati**: anche solo file ruotato in volume Docker, idealmente Loki/CloudWatch/Datadog.

Esempio Node.js con `pino`:

```js
import pino from 'pino';
const log = pino({ level: process.env.LOG_LEVEL || 'info' });
log.info({ user_id: 42, action: 'login' }, 'user logged in');
```

### Metrics

Minimo:

- **Uptime** (è viva?).
- **Request rate** (RPS).
- **Error rate** (% di 5xx).
- **Latency** (p50, p95, p99).

Stack consigliato per solo: **Prometheus + Grafana** self-hosted, oppure **Grafana Cloud free tier**, oppure **Better Stack / Uptime Robot** per just-uptime.

### Traces (opzionale)

Per progetti distribuiti o latenze sospette: **OpenTelemetry** + Jaeger / Tempo / Honeycomb.

### Error tracking

**Obbligatorio** se hai utenti reali:

- **Sentry** — gold standard, free tier generoso.
- **GlitchTip** — fork open-source di Sentry, self-hostabile.
- **Rollbar / Bugsnag** — alternative.

Configura: source maps caricati per stacktrace leggibili, alert email/Slack per nuovi error type, release tracking per associare error a deploy.

### Health check

Endpoint `/health` che restituisce:

```json
{
  "status": "ok",
  "version": "1.2.3",
  "uptime_seconds": 12345,
  "dependencies": {
    "database": "ok",
    "cache": "ok"
  }
}
```

Se una dipendenza è KO, restituisci 503. Health check usato da load balancer e monitoring.

### Alert

Configura alert per:

- Health check rosso > 2 minuti.
- Error rate > 1% per > 5 minuti.
- Latency p95 > soglia per > 5 minuti.
- Disk usage > 80%.
- SSL certificate scadenza < 14 giorni.

Canale: email se solo, Slack/Discord/Telegram personale se vuoi push immediato.

## P3.3 — Database

### Backup automatici

- **Frequenza**: almeno giornaliera per produzione, settimanale per staging.
- **Retention**: 30 giorni daily, 12 mesi weekly, 5 anni monthly (regola "3-2-1": 3 copie, 2 supporti diversi, 1 off-site).
- **Storage**: cloud diverso da quello dell'app (es. app su DigitalOcean, backup su Backblaze B2 o AWS S3).
- **Encryption**: backup cifrato at-rest. Chiave di decrypt nel secret manager, non sul server di backup.

### Test di restore

**Mensile**, su ambiente isolato. Non testato = inesistente.

```bash
# Esempio script di restore test
./scripts/restore-backup.sh --backup-id=2026-05-01 --target=staging-restore
./scripts/verify-restore.sh staging-restore
```

### Migrations

Vedi Fase 4.3.

## P3.4 — SSL/TLS, Domain, DNS

### Certificati

- **Let's Encrypt** via certbot, traefik, o Caddy (Caddy lo gestisce automaticamente, default consigliato).
- **Rinnovo automatico**, mai manuale.
- **Alert** se la scadenza è < 14 giorni: significa che il rinnovo automatico è rotto.

### Domain

- Registra su un registrar **stabile e affidabile** (Cloudflare Registrar, Porkbun, Namecheap). Evita registrar oscuri.
- **2FA** sempre attivo sull'account registrar.
- **Domain lock** attivato.
- **Pagamento auto-renew** attivato, ma **calendar reminder** un mese prima della scadenza come backup.

### DNS

- Provider stabile: Cloudflare DNS (gratis), Route53, DNSimple.
- TTL ragionevoli: 300s per record che potrebbero cambiare (A/CNAME), 3600s per stabili.
- DNSSEC attivato se il provider lo supporta.
- Backup della config DNS (export periodico) versionato.

## P3.5 — Cost Monitoring (cloud)

### Alert sul billing

Su ogni cloud provider:

- Budget alert al 50%, 80%, 100% del previsto.
- Hard limit se possibile (alcuni provider lo permettono, AWS no nativamente).

### Review mensile

Primo del mese, 15 minuti:

- Apri il billing dashboard.
- Confronta col mese precedente.
- Se +20% o più senza ragione nota: investiga.
- Nota in `docs/runbook.md` la spesa mensile baseline.

### Killer comuni

- **Egress** non monitorato (CloudFront/CDN spese a sorpresa).
- **Idle resources** (load balancer dimenticati, EBS volume non collegati, snapshot vecchi).
- **Logs in retention infinita** (CloudWatch può costare più dell'app stessa).
- **Backup snapshot** che non scadono mai.

---

# PARTE IV — Manutenzione Continua

Attività ricorrenti, indipendenti dalle feature. Calendarizzale.

## P4.1 — Settimanale (15-30 min, lunedì mattina)

- Review e merge delle PR Dependabot (vedi A.8.2).
- Triage delle issue: chiudi obsolete, etichetta nuove.
- Check del Kanban: cosa è bloccato e perché?
- Review dei log/error tracking della settimana: nuovi error pattern?

## P4.2 — Mensile (1-2 ore, primo del mese)

- **Cost review** (P3.5).
- **Backup restore test** (P3.3).
- **Security advisory check**: `npm audit`, `pip-audit`, GitHub Security tab.
- **Instinct export e backup** (Parte II, Fase 10.3).
- **Review `docs/tech-debt.md`**: cosa pagare ora che il backlog è vuoto?
- **Aggiornamento documentazione obsoleta** (READMEs, runbook).

## P4.3 — Trimestrale (mezza giornata)

- **Performance test**: carica con k6 / Locust / Apache Bench, confronta con baseline.
  ```bash
  k6 run --vus 50 --duration 5m scripts/load-test.js
  ```
- **Disaster recovery drill**: simula perdita di un componente (DB, server) e ricostruisci da backup.
- **Major dependency review**: ci sono major version pending da troppo? Pianifica upgrade.
- **Review degli ADR**: alcuni sono obsoleti? Marca come "Superseded by ADR-XXXX".
- **Pulizia repository**: branch stale, tag inutili, immagini Docker vecchie nel registry.
- **Renewal check**: domain, certificati, abbonamenti cloud, account services.

## P4.4 — Annuale

- **Rotazione di tutti i secret long-lived**.
- **Review della licenza** del progetto.
- **Audit di security esteso**: penetration test (anche solo OWASP ZAP automatico), audit delle permission cloud.
- **Aggiornamento del runtime principale** (Node N → N+2, Python 3.11 → 3.13, ecc.).
- **Revisione di questo workflow**: cosa hai cambiato in pratica? Aggiorna il documento.

---

# PARTE V — Riferimenti

## V.1 — Branching Strategy

**Trunk-based con short-lived feature branch** (raccomandato per solo):

- `main` è sempre deployabile.
- Feature branch dura **max 1-3 giorni**, poi merge.
- Feature più grandi: spezzane in più PR, usa feature flag per nascondere il work-in-progress.
- Niente long-lived branch (`develop`, `release/*`) — overkill per solo.

Naming:

- `feat/<short-desc>-#<issue>` — nuova feature.
- `fix/<short-desc>-#<issue>` — bug fix.
- `chore/<short-desc>` — manutenzione.
- `refactor/<short-desc>` — refactor.
- `docs/<short-desc>` — solo documentazione.

## V.2 — Conventional Commits

Format:

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

Types:

- `feat` — nuova feature.
- `fix` — bug fix.
- `docs` — documentazione.
- `style` — formatting, no code change.
- `refactor` — refactor senza cambio comportamento.
- `perf` — miglioramento performance.
- `test` — aggiunta/modifica test.
- `chore` — manutenzione, deps.
- `ci` — modifiche CI.
- `build` — modifiche build system.

Breaking change: `feat!:` o footer `BREAKING CHANGE: ...`.

## V.3 — Semantic Versioning

`MAJOR.MINOR.PATCH`:

- **MAJOR**: breaking change incompatibile.
- **MINOR**: nuova funzionalità backward-compatible.
- **PATCH**: bug fix backward-compatible.

Pre-release: `v1.0.0-rc.1`, `v1.0.0-beta.2`, `v1.0.0-alpha.3`.

Pre-1.0: `v0.x.y`. API può cambiare in qualsiasi minor. Quando l'API è stabile, rilascia `v1.0.0`.

## V.4 — ADR Template

```markdown
# ADR XXXX: <titolo>
Stato: Proposto / Accettato / Sostituito da ADR-YYYY / Deprecato
Data: YYYY-MM-DD

## Contesto
Cosa stai decidendo e perché ora. 2-3 paragrafi.

## Opzioni considerate
- A: pro / contro
- B: pro / contro
- C: pro / contro

## Decisione
Quale opzione e perché. 1-2 paragrafi.

## Conseguenze
Cosa cambia, quali rischi accetti, cosa rimandi.

## Condizioni di rivalutazione
Cosa ti farebbe riconsiderare questa decisione (es. "se il numero
di utenti supera 10K", "se il costo cloud supera €500/mese").
```

## V.5 — Spike e Prototipi

- Max 2 ore.
- Domanda specifica documentata in `docs/spikes/YYYY-MM-DD-<domanda>.md`.
- Salta TDD, code review, security review.
- Al termine: scrivi la risposta nel file.
- **Cancella o riscrivi il codice** dello spike. Non promuoverlo a produzione senza Fase 1-7 complete.

## V.6 — Disaster Recovery (regole base)

- **Push su Git** almeno fine giornata.
- **Remote privato su provider diverso** dalle credenziali principali.
- **Backup mensile**: DB, file utente, instinct, file di sessione plugin.
- **Test di restore trimestrale** in ambiente isolato.
- **Password manager** per tutti i secret. **Token di recovery** offline.
- **Hardware backup**: copia degli artefatti critici su SSD esterno crittografato, conservato fuori dalla postazione principale.

## V.7 — Comandi Utili (Plugin)

| Comando | Quando usarlo |
|---|---|
| `/harness-audit` | Verifica config plugin (inizio progetto) |
| `/instinct-status` | Vedere instinct appresi |
| `/instinct-import` | Importare instinct da altro progetto |
| `/instinct-export` | Esportare instinct (backup mensile) |
| `/resume-session` | Riprendere sessione interrotta |
| `/checkpoint` | Punto di ripristino |
| `/build-fix` | Fix automatico errori build |
| `/update-docs` | Aggiornare documentazione |
| `/loop-start` | Loop iterativo su task |
| `/model-route` | Scegliere modello ottimale |
| `/skill-create` | Nuova skill personalizzata |
| `/multi-execute` | Task in parallelo su più fronti |
| `/multi-plan` | Pianificazione multi-fronte |
| `/verification-loop` | Verification completo pre-PR |
| `/security-review` | Security review strutturata |
| `/quality-gate` | Quality gate intermedio |
| `/test-coverage` | Verificare coverage |
| `/code-review` | Code review generica |
| `/python-review` / `/go-review` / `/rust-review` / ... | Review per linguaggio |
| `/refactor-clean` | Refactor mirato (timebox 1h) |
| `/council` | Decisione architetturale strutturata |
| `/prp-commit` | Conventional commit |
| `/prp-pr` | Generazione PR |
| `/review-pr` | Review PR esistente |
| `/learn` | Estrai pattern dalla sessione |
| `/save-session` | Salva contesto sessione |
| `/evolve` | Evolvi instinct |
| `/compact` (nativo Claude Code) | Pulisce il contesto |

## V.8 — Comandi Utili (CLI sistema)

| Tool | Uso |
|---|---|
| `npx ecc-agentshield scan` | Security scan completo |
| `trivy image <img>` | Scan vulnerabilità immagine Docker |
| `gitleaks detect` | Trova secret nel repo |
| `npm audit` / `pip-audit` | Audit dipendenze |
| `git log --oneline --graph` | Storia branch |
| `git rebase -i origin/main` | Pulizia commit history |
| `git filter-repo` | Riscrittura storia (rimuovere secret) |
| `docker compose up -d` | Start ambiente locale |
| `k6 run scripts/load.js` | Load test |

## V.9 — Tabella Riassuntiva: Cosa Fare in Ogni Fase

| Fase | Output | Comando chiave | Salta se |
|---|---|---|---|
| 0 — Scoping | Issue GitHub, branch, ADR se serve | `/harness-audit` | MAI |
| 1 — Search-first | Confronto librerie, licenze ok | `search-first` | MAI |
| 2 — Council | ADR di decisione | `/council` | Decisione facilmente reversibile |
| 3 — Pianificazione | Task list nella issue o `docs/plan.md` | `/multi-plan`, `/compact` | MAI |
| 4 — Sviluppo | Codice + test, checkpoint, commit | `/checkpoint`, `/test-coverage` | MAI (TDD adattivo) |
| 5 — Code review | Review applicata, debito tracciato | `/code-review`, `/refactor-clean` | MAI |
| 6 — Security | Scan PASS, eccezioni documentate | `/security-review` + `agentshield` + `trivy` | Solo se locale single-user senza rete |
| 7 — Verifica | Verification loop PASS | `/verification-loop` | MAI |
| 8 — Git & PR | PR aperta, CI verde, merge squash | `/prp-commit`, `/prp-pr`, `/review-pr` | MAI |
| 9 — Docs | README, CHANGELOG, ADR aggiornati nello stesso PR | `/update-docs` | MAI |
| 10 — Chiusura | Sessione salvata, push | `/learn`, `/save-session`, `/evolve` | MAI |

## V.10 — Quick Reference: Sequenza Standard di una Feature

Setup iniziale (one-time, Parte I) presunto fatto. Per una feature media:

```
1.  git checkout main && git pull && git checkout -b feat/<desc>-#<issue>
2.  /compact "Focus on implementing <feature>"
3.  Implementa <feature> con tdd-workflow, coverage target X%
4.  /checkpoint dopo ogni step verde
5.  /test-coverage
6.  /code-review (o variante linguaggio)
7.  /refactor-clean (se serve, timebox 1h)
8.  Se security-relevant: /security-review + npx ecc-agentshield scan + trivy
9.  /quality-gate
10. /update-docs
11. /prp-commit
12. git rebase origin/main
13. git push origin feat/<branch>
14. /prp-pr (apri PR su GitHub)
15. /review-pr
16. /verification-loop
17. Aspetta CI verde
18. Squash and merge
19. Se milestone: tag v1.X.Y, GitHub Release
20. Deploy seguendo Parte III
```

A fine giornata:

```
21. /learn
22. /save-session
23. /evolve
24. git push (anche WIP)
```

A fine milestone:

```
25. Aggiorna CHANGELOG.md con la versione
26. Tag e release
27. Deploy production seguendo Parte III pre-deploy checklist
```

## V.11 — Anti-Pattern da Evitare

1. Saltare Parte I "perché è un progetto piccolo".
2. Coverage 80% rigido ovunque.
3. TDD per spike.
4. Refactor "veloce" senza timebox.
5. Saltare security review "perché è solo per me".
6. Documentare dopo.
7. Backup mai testati.
8. Merge diretto su main senza PR.
9. Instinct mai esportati.
10. Fidarsi di `/code-review` come unica review.
11. Secret in `Dockerfile` "tanto è per dev".
12. `latest` tag per immagini Docker in production.
13. Major dependency upgrade auto-merged.
14. `--no-verify` su commit destinati a main.
15. Long-lived feature branch (> 3 giorni).
16. Migration irreversibili senza piano in due step.
17. Deploy senza pre-deploy checklist.
18. Alert configurati ma mai testati (silent failure).
19. Cost review "la farò il mese prossimo" per 6 mesi.
20. Rinnovo dominio/certificato senza calendar reminder backup.

## V.12 — Manutenzione di Questo Documento

Questo workflow non è scolpito nella pietra. Se una fase ti rallenta sistematicamente, **non saltarla silenziosamente**: aggiorna il documento spiegando cosa hai cambiato e perché. Aggiungi una entry nel CHANGELOG personale del workflow.

**Revisione consigliata**: ogni 3 mesi, o dopo ogni cambio significativo di stack/infrastruttura. Trattalo come codice: versionato, modificato con consapevolezza, mai abbandonato.
