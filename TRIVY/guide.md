# Trivy — Guida Completa: Installazione, Setup e Uso Professionale su WSL2 Ubuntu

> Guida pratica e ragionata per passare da zero a un utilizzo professionale di **Trivy**, lo scanner di vulnerabilità open-source di Aqua Security. Pensata per chi parte senza alcuna esperienza ma vuole capire **cosa fa**, **perché**, e **come si usa nel mondo reale** (DevSecOps, CI/CD, hardening, audit).

---

## Indice

1. [Cos'è Trivy e perché usarlo](#1-cosè-trivy-e-perché-usarlo)
2. [Cosa scansiona Trivy (in dettaglio)](#2-cosa-scansiona-trivy-in-dettaglio)
3. [Architettura: come funziona sotto il cofano](#3-architettura-come-funziona-sotto-il-cofano)
4. [Prerequisiti: preparare WSL2 Ubuntu](#4-prerequisiti-preparare-wsl2-ubuntu)
5. [Installazione di Trivy su Ubuntu (WSL2)](#5-installazione-di-trivy-su-ubuntu-wsl2)
6. [Verifica installazione e prima esecuzione](#6-verifica-installazione-e-prima-esecuzione)
7. [Database delle vulnerabilità: gestione e cache](#7-database-delle-vulnerabilità-gestione-e-cache)
8. [I cinque target principali (target types)](#8-i-cinque-target-principali-target-types)
9. [Scansione di immagini Docker/Container](#9-scansione-di-immagini-dockercontainer)
10. [Scansione di filesystem e progetti locali](#10-scansione-di-filesystem-e-progetti-locali)
11. [Scansione di repository Git](#11-scansione-di-repository-git)
12. [Scansione IaC e Misconfigurations (Terraform, Kubernetes, Dockerfile…)](#12-scansione-iac-e-misconfigurations)
13. [Rilevamento di Secrets](#13-rilevamento-di-secrets)
14. [Rilevamento di licenze](#14-rilevamento-di-licenze)
15. [Generazione di SBOM (CycloneDX, SPDX)](#15-generazione-di-sbom-cyclonedx-spdx)
16. [Scansione di cluster Kubernetes](#16-scansione-di-cluster-kubernetes)
17. [Filtri, severity, exit code, ignore list](#17-filtri-severity-exit-code-ignore-list)
18. [File di configurazione `trivy.yaml`](#18-file-di-configurazione-trivyyaml)
19. [Output e report (JSON, SARIF, HTML, tabella)](#19-output-e-report-json-sarif-html-tabella)
20. [Integrazione in CI/CD (GitHub Actions, GitLab, pre-commit)](#20-integrazione-in-cicd)
21. [Modalità Server/Client (per team e CI distribuita)](#21-modalità-serverclient)
22. [Aggiornamenti, manutenzione e disinstallazione](#22-aggiornamenti-manutenzione-e-disinstallazione)
23. [Troubleshooting su WSL2](#23-troubleshooting-su-wsl2)
24. [Best practice da professionista](#24-best-practice-da-professionista)
25. [Riferimenti](#25-riferimenti)

---

## 1. Cos'è Trivy e perché usarlo

**Trivy** (pronunciato *tri-vy*) è uno scanner di sicurezza **open-source**, sviluppato da **Aqua Security**, distribuito sotto licenza Apache 2.0. È diventato uno strumento standard nell'industria DevSecOps perché:

- È **completo**: in un singolo binario unifica funzionalità che richiederebbero molti tool separati (vulnerabilità, IaC, secrets, licenze, SBOM, Kubernetes).
- È **veloce**: non richiede agenti, non installa nulla nei container, e usa un database di vulnerabilità precompilato e indicizzato.
- È **affidabile**: aggrega dati da fonti autorevoli (NVD, GitHub Security Advisories, vendor advisories di Red Hat, Debian, Ubuntu, Alpine, Amazon Linux, ecc.).
- È **CI-friendly**: produce output machine-readable (JSON, SARIF, CycloneDX, SPDX) e si integra in qualunque pipeline.
- È **gratuito** e maintenuto attivamente.

In pratica Trivy risponde a domande come:

- "Questa immagine Docker che sto per pubblicare in produzione contiene CVE critici?"
- "Il mio `package.json` / `go.mod` / `pom.xml` / `requirements.txt` ha dipendenze vulnerabili?"
- "Il mio Terraform o Helm chart espone un S3 bucket pubblico o un container `privileged: true`?"
- "C'è un AWS access key dimenticato in un commit?"
- "Quali licenze usano le mie dipendenze e sono compatibili con il mio prodotto?"
- "Posso generare un SBUM (Software Bill of Materials) per compliance?"

---

## 2. Cosa scansiona Trivy (in dettaglio)

| Categoria | Esempi |
|---|---|
| **Vulnerabilità OS** | pacchetti di Alpine, Debian, Ubuntu, RHEL/CentOS, Amazon Linux, SUSE, Photon, Oracle Linux, Wolfi, Chainguard. |
| **Vulnerabilità linguaggio** | npm/yarn/pnpm (Node.js), pip/poetry/pipenv (Python), Go modules, Cargo (Rust), Maven/Gradle (Java), NuGet (.NET), Composer (PHP), gem (Ruby), Bun, Conan (C/C++). |
| **Misconfigurations (IaC)** | Terraform, CloudFormation, Kubernetes manifest, Helm chart, Dockerfile, Docker Compose, Ansible (limitato), Azure ARM. |
| **Secrets** | AWS keys, GCP keys, GitHub/GitLab token, Slack token, chiavi private SSH/PGP, JWT, password hardcoded, ecc. |
| **Licenze** | Riconoscimento automatico delle licenze di OS package e dipendenze applicative. |
| **SBOM** | Generazione e scansione di SBOM in formato CycloneDX e SPDX. |
| **Kubernetes** | Cluster live (vulnerabilità delle immagini in esecuzione + misconfigurations dei manifest). |

---

## 3. Architettura: come funziona sotto il cofano

Capire l'architettura aiuta a usarlo bene e a fare debug. In sintesi:

1. **Analizzatori (analyzers)**: Trivy ispeziona il target (immagine, filesystem, repo, ecc.) e identifica artefatti rilevanti — file di lock, manifest di pacchetti OS, manifest IaC, file binari.
2. **Estrazione SBOM interna**: per ogni artefatto produce internamente una lista di componenti con versioni precise.
3. **Database delle vulnerabilità**: un DB precompilato (`trivy-db`) viene scaricato come **OCI artifact** dal registry pubblico (di default `ghcr.io/aquasecurity/trivy-db`).
4. **Matching**: i componenti rilevati vengono incrociati con il DB → escono i CVE.
5. **Policy engine**: per IaC e misconfigurations Trivy usa **Rego/OPA** internamente con un set di policy mantenute dalla community (in passato chiamate "trivy-checks", originariamente "tfsec policies").
6. **Reportistica**: l'output passa attraverso formattatori (table, json, sarif, html, cyclonedx, spdx-json…).

I database principali che Trivy gestisce localmente:

- `trivy-db` → vulnerabilità (CVE).
- `trivy-java-db` → vulnerabilità Java/Maven (separato perché molto grande).
- `trivy-checks` → policy IaC e misconfigurations.

Vengono salvati in cache in `~/.cache/trivy/`.

---

## 4. Prerequisiti: preparare WSL2 Ubuntu

Trivy gira nativamente su Linux ed è perfetto in WSL2. Prima di installarlo conviene avere un ambiente pulito.

### 4.1 Verifica di essere su WSL2 e su Ubuntu

Apri il terminale Ubuntu (es. da menu Start → "Ubuntu") e lancia:

```bash
# Verifica distribuzione
lsb_release -a

# Verifica kernel WSL2
uname -r
# Output tipico WSL2: 5.x.x.x-microsoft-standard-WSL2 oppure 6.x.x...-microsoft-standard-WSL2
```

Se sei ancora su WSL1, da PowerShell (Windows) come amministratore:

```powershell
wsl --set-version Ubuntu 2
wsl --set-default-version 2
```

### 4.2 Aggiorna il sistema

```bash
sudo apt update && sudo apt upgrade -y
```

### 4.3 Installa pacchetti di supporto

```bash
sudo apt install -y wget curl gnupg lsb-release apt-transport-https ca-certificates software-properties-common
```

### 4.4 (Opzionale ma consigliato) Installa Docker su WSL2

Trivy può scansionare immagini direttamente da un registry remoto, ma per molti workflow è comodo avere Docker locale. Le opzioni:

- **Docker Desktop** su Windows con integrazione WSL2 attiva (opzione più semplice).
- **Docker Engine nativo** dentro Ubuntu WSL2:

```bash
# Installazione Docker Engine ufficiale
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Aggiungi il tuo utente al gruppo docker (così non serve sudo)
sudo usermod -aG docker $USER

# Avvia il servizio (in WSL2 systemd va abilitato; alternativa: dockerd manuale)
sudo service docker start
```

Per abilitare `systemd` in WSL2 (Windows 11 + versioni recenti di WSL):

```bash
# /etc/wsl.conf
sudo tee /etc/wsl.conf > /dev/null <<'EOF'
[boot]
systemd=true
EOF
```

Poi da PowerShell: `wsl --shutdown` e riapri Ubuntu.

> **Nota**: Trivy **non richiede Docker** per funzionare. Può scansionare immagini "remote" via API del registry, filesystem, repository Git, ecc. Docker è utile ma facoltativo.

---

## 5. Installazione di Trivy su Ubuntu (WSL2)

Esistono 4 modi principali. In ordine di "professionalità":

### 5.1 Metodo consigliato: repository APT ufficiale di Aqua Security

Questo è il metodo standard in produzione: avrai aggiornamenti gestiti da `apt` insieme al resto del sistema.

```bash
# 1. Aggiungi la chiave GPG ufficiale
sudo install -m 0755 -d /etc/apt/keyrings
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key \
  | sudo gpg --dearmor -o /etc/apt/keyrings/trivy.gpg

# 2. Aggiungi il repository
echo "deb [signed-by=/etc/apt/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" \
  | sudo tee /etc/apt/sources.list.d/trivy.list

# 3. Aggiorna l'indice e installa
sudo apt update
sudo apt install -y trivy
```

### 5.2 Installer ufficiale (one-liner)

Utile in container minimal o in CI runner senza root manager pacchetti complessi:

```bash
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh \
  | sudo sh -s -- -b /usr/local/bin
```

Lo script scarica l'ultima release dal GitHub di Trivy e la mette in `/usr/local/bin/trivy`.

### 5.3 Binario pre-compilato dalla pagina Releases

Per ambienti molto controllati (air-gapped) puoi scaricare manualmente:

```bash
# Esempio: ultima release per linux/amd64
TRIVY_VERSION=$(curl -s https://api.github.com/repos/aquasecurity/trivy/releases/latest \
  | grep -Po '"tag_name": "v\K[^"]+')
wget https://github.com/aquasecurity/trivy/releases/download/v${TRIVY_VERSION}/trivy_${TRIVY_VERSION}_Linux-64bit.deb
sudo dpkg -i trivy_${TRIVY_VERSION}_Linux-64bit.deb
```

### 5.4 Tramite immagine Docker (no install)

Se preferisci non installare nulla:

```bash
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  -v $HOME/.cache/trivy:/root/.cache/ \
  aquasec/trivy:latest image alpine:3.18
```

> Per uso "professionale" su WSL2 consiglio il **metodo 5.1** (APT): è quello che si aggiorna in modo pulito e si integra con `unattended-upgrades`.

---

## 6. Verifica installazione e prima esecuzione

```bash
# Versione (dovrebbe stampare la release installata)
trivy --version

# Help generale
trivy --help

# Help di un sottocomando
trivy image --help
```

### Prima scansione di prova

Una scansione "hello world" è un'immagine pubblica piccola e nota:

```bash
trivy image alpine:3.18
```

Cosa succede al primo lancio:

1. Trivy scarica `trivy-db` (qualche decina di MB) in `~/.cache/trivy/`.
2. Estrae i pacchetti dell'immagine `alpine:3.18`.
3. Confronta con il DB.
4. Stampa una tabella con CVE, severity, pacchetto, versione installata, versione fixed.

Esempio di output (semplificato):

```
alpine:3.18 (alpine 3.18.x)
==========================
Total: N (UNKNOWN: 0, LOW: x, MEDIUM: y, HIGH: z, CRITICAL: w)

┌─────────────┬────────────────┬──────────┬───────────┬─────────────┬────────────────────────────┐
│   Library   │ Vulnerability  │ Severity │ Installed │   Fixed     │           Title            │
├─────────────┼────────────────┼──────────┼───────────┼─────────────┼────────────────────────────┤
│ openssl     │ CVE-2024-XXXX  │ HIGH     │ 3.x.x     │ 3.x.y       │ openssl: ...               │
└─────────────┴────────────────┴──────────┴───────────┴─────────────┴────────────────────────────┘
```

---

## 7. Database delle vulnerabilità: gestione e cache

### 7.1 Dove sono i dati

```bash
ls -lah ~/.cache/trivy/
# db/ → trivy-db
# java-db/ → solo se hai scansionato JAR/Maven
# fanal/ → cache dei layer/artefatti analizzati
```

### 7.2 Aggiornamento del DB

Di default Trivy aggiorna il DB **automaticamente** se vede che è obsoleto (TTL di 24 ore). Puoi forzare manualmente:

```bash
# Aggiorna solo il DB (senza fare scansione)
trivy image --download-db-only

# Aggiorna anche il DB Java (utile prima di scansionare progetti Java offline)
trivy image --download-java-db-only
```

### 7.3 Skip aggiornamento (utile in CI o offline)

```bash
trivy image --skip-db-update --skip-java-db-update myimage:tag
```

In CI è buona pratica fare un job dedicato che aggiorna il DB e lo cacha, in modo da non scaricarlo a ogni job.

### 7.4 Modalità air-gapped

Per ambienti senza Internet:

```bash
# Su una macchina online
trivy image --download-db-only
# Copia ~/.cache/trivy/db/ sulla macchina target

# Sulla macchina offline
trivy image --skip-db-update --offline-scan ...
```

---

## 8. I cinque target principali (target types)

Trivy si invoca così:

```
trivy <target> [opzioni] <argomento>
```

I target più usati:

| Comando | Cosa scansiona |
|---|---|
| `trivy image`        | un'immagine container (locale o da registry) |
| `trivy filesystem` (`fs`) | una directory locale (codice + lockfile) |
| `trivy repository` (`repo`) | un repo Git remoto |
| `trivy config`       | file IaC (Terraform, K8s, Dockerfile, Helm, …) |
| `trivy sbom`         | un SBOM esistente (CycloneDX/SPDX) |
| `trivy kubernetes` (`k8s`) | un cluster Kubernetes attivo |
| `trivy rootfs`       | un filesystem estratto (senza metadati immagine) |
| `trivy vm`           | immagini di VM (AMI, VMDK, …) |

---

## 9. Scansione di immagini Docker/Container

### 9.1 Immagine pubblica da Docker Hub

```bash
trivy image nginx:1.27
```

### 9.2 Immagine locale (già `docker pull`-ata o `docker build`-ata)

```bash
docker build -t mia-app:dev .
trivy image mia-app:dev
```

### 9.3 Solo vulnerabilità di severity alta o critica

```bash
trivy image --severity HIGH,CRITICAL nginx:1.27
```

### 9.4 Solo CVE con fix disponibile (azionabili subito)

```bash
trivy image --ignore-unfixed --severity HIGH,CRITICAL nginx:1.27
```

### 9.5 Scansione con secrets e misconfig in un colpo solo

```bash
trivy image --scanners vuln,secret,misconfig nginx:1.27
```

### 9.6 Da registry privato (con login Docker)

```bash
docker login mia-registry.example.com
trivy image mia-registry.example.com/team/app:1.2.3
```

### 9.7 Da tarball (immagine esportata)

```bash
docker save -o app.tar mia-app:dev
trivy image --input app.tar
```

---

## 10. Scansione di filesystem e progetti locali

Utile mentre sviluppi: scansiona la cartella del progetto, lockfile inclusi.

```bash
# Esempio: progetto Node
cd ~/progetti/mia-app
trivy fs .

# Solo vulnerabilità (escludi misconfig/secret)
trivy fs --scanners vuln .

# Limita ai file di lock conosciuti (più veloce)
trivy fs --scanners vuln --severity HIGH,CRITICAL .
```

Trivy riconosce automaticamente:

- `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`
- `requirements.txt`, `Pipfile.lock`, `poetry.lock`, `uv.lock`
- `go.mod`, `go.sum`
- `Cargo.lock`
- `pom.xml`, `gradle.lockfile`
- `composer.lock`
- `Gemfile.lock`

---

## 11. Scansione di repository Git

Stesso motore del filesystem ma con clone diretto, comodo per audit di repo che non hai ancora in locale:

```bash
# Repo pubblico
trivy repo https://github.com/aquasecurity/trivy-ci-test

# Branch specifico
trivy repo --branch develop https://github.com/org/repo

# Singolo commit / tag
trivy repo --commit <sha> https://github.com/org/repo
trivy repo --tag v1.2.3 https://github.com/org/repo
```

---

## 12. Scansione IaC e Misconfigurations

Questa è una delle feature più potenti: Trivy controlla che il tuo Terraform, i tuoi manifest Kubernetes, i tuoi Dockerfile, i tuoi Helm chart, ecc., rispettino best practice di sicurezza note.

```bash
# Scansiona tutta la directory in cerca di misconfig
trivy config .

# Solo Terraform / solo K8s / solo Dockerfile (Trivy auto-rileva ma puoi filtrare)
trivy config --misconfig-scanners terraform .
trivy config --misconfig-scanners kubernetes .
trivy config --misconfig-scanners dockerfile .

# Severity minima
trivy config --severity HIGH,CRITICAL .
```

Esempi di check tipici:

- Dockerfile che gira come `root`.
- Container Kubernetes senza `resources.limits`.
- `privileged: true` o `hostNetwork: true`.
- S3 bucket Terraform senza encryption at rest.
- Security group AWS che apre `0.0.0.0/0` su porta SSH.
- Ingress senza TLS.

---

## 13. Rilevamento di Secrets

```bash
# Scansiona file e/o codice in cerca di secret hardcoded
trivy fs --scanners secret .

# Stessa cosa su un'immagine
trivy image --scanners secret nginx:1.27
```

Esempi di pattern rilevati: AWS access key, GitHub PAT, Slack webhook, chiavi private, JWT, password in stringa.

> Trivy può generare falsi positivi (es. esempi nei test). Usa la sezione [Filtri e ignore list](#17-filtri-severity-exit-code-ignore-list) per gestirli.

---

## 14. Rilevamento di licenze

Utile per compliance legale / open source governance.

```bash
# Scansione filesystem con licenze
trivy fs --scanners license .

# Scansione immagine con licenze classificate per rischio
trivy image --scanners license --license-full nginx:1.27
```

Trivy classifica le licenze in livelli (`FORBIDDEN`, `RESTRICTED`, `RECIPROCAL`, `NOTICE`, `PERMISSIVE`, `UNENCUMBERED`, `UNKNOWN`) sulla base delle linee guida Google open source.

---

## 15. Generazione di SBOM (CycloneDX, SPDX)

Un SBOM è la "lista degli ingredienti" del tuo software. È sempre più richiesto per compliance (es. Executive Order USA 14028, CRA europeo).

```bash
# CycloneDX JSON
trivy image --format cyclonedx --output sbom.cdx.json nginx:1.27

# SPDX JSON
trivy image --format spdx-json --output sbom.spdx.json nginx:1.27

# SPDX classico
trivy image --format spdx --output sbom.spdx nginx:1.27
```

Una volta che hai un SBOM puoi rifare scan a partire da quello (più veloce, niente bisogno dell'immagine):

```bash
trivy sbom sbom.cdx.json
```

---

## 16. Scansione di cluster Kubernetes

Per usare questa modalità su WSL2 ti basta avere un `kubectl` configurato (es. con `~/.kube/config` che punta al cluster).

```bash
# Panoramica del cluster (workload + misconfig)
trivy k8s --report summary cluster

# Tutti i namespace, output dettagliato
trivy k8s --report all cluster

# Solo un namespace
trivy k8s --include-namespaces production cluster

# Solo vulnerabilità delle immagini in esecuzione
trivy k8s --scanners vuln cluster
```

---

## 17. Filtri, severity, exit code, ignore list

### 17.1 Severity

```bash
trivy image --severity CRITICAL,HIGH nginx:1.27
```

### 17.2 Far fallire la pipeline

In CI vuoi che `trivy` ritorni exit-code ≠ 0 se trova qualcosa di critico:

```bash
trivy image --severity CRITICAL --exit-code 1 nginx:1.27
```

### 17.3 Ignorare CVE specifici (`.trivyignore`)

Crea un file `.trivyignore` nella root del progetto:

```text
# Falso positivo: il modulo non viene caricato
CVE-2023-12345

# Mitigato a livello di network policy fino al 2026-01-01
CVE-2024-99999 exp:2026-01-01
```

E lancia normalmente: Trivy lo legge automaticamente.

### 17.4 Ignorare via policy avanzata (`--ignore-policy`)

Per regole più ricche puoi scrivere policy Rego:

```bash
trivy image --ignore-policy ignore.rego nginx:1.27
```

### 17.5 Saltare path

```bash
trivy fs --skip-dirs node_modules --skip-files "**/test/**" .
```

---

## 18. File di configurazione `trivy.yaml`

Nel tuo progetto, anziché lunghissime righe di flag, crea `trivy.yaml`:

```yaml
# trivy.yaml
severity:
  - HIGH
  - CRITICAL

scan:
  scanners:
    - vuln
    - secret
    - misconfig
  skip-dirs:
    - node_modules
    - vendor
    - .venv

vulnerability:
  ignore-unfixed: true

misconfiguration:
  include-non-failures: false

format: table
exit-code: 1
```

Poi:

```bash
trivy fs --config trivy.yaml .
```

> Convenzione professionale: committa `trivy.yaml` nel repo e referenzialo dalla pipeline CI così tutti scansionano con gli stessi criteri.

---

## 19. Output e report (JSON, SARIF, HTML, tabella)

### 19.1 Tabella (default)

```bash
trivy image nginx:1.27
```

### 19.2 JSON

```bash
trivy image --format json --output report.json nginx:1.27
```

### 19.3 SARIF (per GitHub Code Scanning)

```bash
trivy image --format sarif --output trivy.sarif nginx:1.27
```

### 19.4 HTML (con template)

Trivy fornisce template Go pronti:

```bash
trivy image \
  --format template \
  --template "@/usr/local/share/trivy/templates/html.tpl" \
  --output report.html \
  nginx:1.27
```

Su Ubuntu via APT i template stanno in `/usr/share/trivy/templates/` o `/usr/local/share/trivy/templates/`. Verifica con:

```bash
find / -name "html.tpl" 2>/dev/null
```

### 19.5 JUnit (per CI con visualizzazione test)

```bash
trivy image --format template --template "@contrib/junit.tpl" --output junit.xml nginx:1.27
```

---

## 20. Integrazione in CI/CD

### 20.1 GitHub Actions

```yaml
# .github/workflows/security.yml
name: security

on: [push, pull_request]

jobs:
  trivy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run Trivy filesystem scan
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: fs
          scan-ref: .
          severity: HIGH,CRITICAL
          exit-code: 1
          format: sarif
          output: trivy.sarif

      - name: Upload SARIF to GitHub Security
        if: always()
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: trivy.sarif
```

### 20.2 GitLab CI

```yaml
# .gitlab-ci.yml
trivy:
  image:
    name: aquasec/trivy:latest
    entrypoint: [""]
  script:
    - trivy fs --severity HIGH,CRITICAL --exit-code 1 .
```

### 20.3 Pre-commit hook locale

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/aquasecurity/trivy
    rev: v0.55.0
    hooks:
      - id: trivy-config
      - id: trivy-fs
```

---

## 21. Modalità Server/Client

In team, far scaricare il DB a ogni sviluppatore è uno spreco. Trivy ha una modalità server:

**Server** (es. su una VM interna):

```bash
trivy server --listen 0.0.0.0:4954
```

**Client** (su WSL2, sviluppatori, runner CI):

```bash
trivy image --server http://trivy-server.interno:4954 nginx:1.27
```

Vantaggi: cache centralizzata, controllo unico delle versioni del DB, meno banda.

---

## 22. Aggiornamenti, manutenzione e disinstallazione

### 22.1 Aggiornare (via APT)

```bash
sudo apt update
sudo apt upgrade trivy
```

### 22.2 Pulire la cache

```bash
trivy clean --all
# oppure manualmente
rm -rf ~/.cache/trivy
```

### 22.3 Disinstallare

```bash
sudo apt remove --purge trivy
sudo rm -f /etc/apt/sources.list.d/trivy.list
sudo rm -f /etc/apt/keyrings/trivy.gpg
rm -rf ~/.cache/trivy
```

---

## 23. Troubleshooting su WSL2

### 23.1 "permission denied" sul socket Docker

```bash
sudo usermod -aG docker $USER
newgrp docker
```

### 23.2 DNS lento al primo download del DB

```bash
# /etc/resolv.conf in WSL2 a volte punta a un IP virtuale lento. Forza DNS pubblici:
sudo tee /etc/wsl.conf > /dev/null <<'EOF'
[network]
generateResolvConf = false
EOF

sudo rm /etc/resolv.conf
echo "nameserver 1.1.1.1" | sudo tee /etc/resolv.conf
echo "nameserver 8.8.8.8" | sudo tee -a /etc/resolv.conf
sudo chattr +i /etc/resolv.conf
```

Poi `wsl --shutdown` da PowerShell.

### 23.3 "TOOMANYREQUESTS" dal registry GitHub Container Registry quando scarica `trivy-db`

Capita su CI/condivisione IP. Soluzioni:

- Aspetta e riprova (rate limit orario).
- Configura mirror interno con `trivy server`.
- Autenticati via `--db-repository` puntando a un mirror privato.

### 23.4 Scansione lentissima su filesystem grandi

```bash
trivy fs --skip-dirs node_modules --skip-dirs .git --skip-dirs dist .
```

### 23.5 Memoria insufficiente in WSL2

Modifica `%UserProfile%\.wslconfig` su Windows:

```ini
[wsl2]
memory=8GB
processors=4
swap=4GB
```

Poi `wsl --shutdown`.

### 23.6 Errore di certificato dietro proxy aziendale

```bash
export HTTPS_PROXY=http://proxy.azienda.local:8080
export HTTP_PROXY=http://proxy.azienda.local:8080
export NO_PROXY=localhost,127.0.0.1
# Se serve un CA custom:
export SSL_CERT_FILE=/etc/ssl/certs/ca-certificates.crt
```

---

## 24. Best practice da professionista

1. **Versiona la configurazione**: `trivy.yaml` + `.trivyignore` nel repo, in modo che la scansione sia riproducibile.
2. **Pinna la severity policy**: blocca la build solo su `HIGH,CRITICAL` ma logga tutto (`MEDIUM` e `LOW` come warning).
3. **Usa `--ignore-unfixed` in fase iniziale**: ti concentri sui CVE con fix disponibile, riduci rumore e mostri ROI immediato.
4. **Genera sempre un SBOM** e archivialo come artifact della build (compliance e tracciabilità).
5. **Esegui la scansione in più punti**:
   - in locale come `pre-commit` (feedback immediato),
   - in CI sul filesystem e sull'immagine prima della pubblicazione,
   - in CD sui cluster Kubernetes (drift e nuove CVE).
6. **Cache del DB in CI**: salva `~/.cache/trivy` come cache della pipeline, eviti rate limit e velocizzi.
7. **Centralizza con server mode** se il team è > 5 persone.
8. **Documenta le `.trivyignore`**: ogni riga deve avere un commento con motivo, owner e data di revisione (`exp:`).
9. **Integra SARIF con GitHub Code Scanning o GitLab Vulnerability Report**: i risultati diventano azionabili dentro la code review.
10. **Aggiorna Trivy regolarmente**: nuovi analyzer e fix di falsi positivi escono frequentemente.

---

## 25. Riferimenti

- Sito ufficiale: https://trivy.dev
- Repository GitHub: https://github.com/aquasecurity/trivy
- Documentazione: https://aquasecurity.github.io/trivy
- Database vulnerabilità: https://github.com/aquasecurity/trivy-db
- Policy IaC: https://github.com/aquasecurity/trivy-checks
- GitHub Action: https://github.com/aquasecurity/trivy-action

---

### Riepilogo rapido (cheat sheet)

```bash
# Installazione (WSL2 Ubuntu)
sudo install -m 0755 -d /etc/apt/keyrings
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo gpg --dearmor -o /etc/apt/keyrings/trivy.gpg
echo "deb [signed-by=/etc/apt/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/trivy.list
sudo apt update && sudo apt install -y trivy

# Scansione immagine
trivy image --severity HIGH,CRITICAL --ignore-unfixed nginx:1.27

# Scansione progetto locale (codice + IaC + secrets)
trivy fs --scanners vuln,secret,misconfig --severity HIGH,CRITICAL .

# Scansione repo Git remoto
trivy repo https://github.com/org/repo

# IaC dedicato
trivy config --severity HIGH,CRITICAL ./infra

# SBOM CycloneDX
trivy image --format cyclonedx -o sbom.cdx.json myimg:tag

# Cluster Kubernetes
trivy k8s --report summary cluster

# Aggiorna solo il DB
trivy image --download-db-only

# Pulisci cache
trivy clean --all
```

> Con questa guida hai tutto il necessario per usare Trivy come un professionista: dall'installazione pulita su WSL2, all'integrazione in CI/CD, fino alla gestione di SBOM e compliance. Da qui in poi è solo questione di abituarti ai comandi e adattare `trivy.yaml` ai tuoi progetti.
