# Guida professionale all'installazione e configurazione di Docker su Windows + WSL2 Ubuntu

Questa guida è pensata per chi non ha **mai** usato Docker e vuole partire da zero, ottenendo un ambiente di lavoro **professionale, sicuro e pronto alla produzione**. È scritta per Windows 10/11 con WSL2 e Ubuntu già installato.

Alla fine avrai:

- Docker Engine + Docker CLI + Docker Compose v2 + BuildKit funzionanti dentro WSL2
- Integrazione completa con Windows (puoi lanciare `docker` dal terminale Ubuntu)
- Configurazione del daemon ottimizzata (logging, storage, sicurezza)
- Networking WSL2 ottimizzato (incluso il nuovo `mirrored` mode)
- Gestione utente senza `sudo`
- Workflow di sviluppo con Dev Containers (VS Code)
- Buone pratiche di sicurezza (SBOM, provenance, digest pinning, scansione CVE)
- Un primo container di test, un Dockerfile multi-stage e un `docker-compose.yml` reali
- Hot-reload con `compose watch`, profili per ambienti dev/prod, GPU passthrough opzionale

---

## Indice

1. Cos'è Docker, in due parole
2. Prerequisiti e verifiche iniziali
3. Configurazione di WSL2 e di Windows
4. Quale Docker installare? Due strade possibili
5. Opzione A — Docker Desktop con backend WSL2
6. Opzione B — Docker Engine nativo dentro WSL2 Ubuntu
7. Configurazione professionale del daemon (`daemon.json`)
8. Permessi: usare Docker senza `sudo`
9. Verifica finale dell'installazione
10. Performance: dove tenere i progetti
11. Struttura di progetto consigliata
12. `Dockerfile` professionale (multi-stage, non-root, healthcheck, lint)
13. `docker-compose.yml` professionale (limiti, healthcheck, profili, watch)
14. Reti e volumi
15. Gestione di variabili d'ambiente e secret
16. GPU passthrough NVIDIA (opzionale)
17. Dev Containers con VS Code
18. `docker context`: gestire più ambienti
19. Manutenzione: comandi quotidiani
20. Sicurezza: hardening, SBOM, provenance, scansione
21. Rootless mode (avanzato, opzionale)
22. Aggiornamenti
23. Backup dei dati persistenti
24. Logging centralizzato
25. Troubleshooting rapido
26. Prossimi passi consigliati
27. Checklist del professionista

---

## 1. Cos'è Docker, in due parole

Docker è una piattaforma per **containerizzare** applicazioni. Un container è un processo isolato che gira su un kernel Linux condiviso, ma con il proprio filesystem, le proprie dipendenze e la propria rete. Questo ti permette di:

- avere ambienti riproducibili ("funziona sulla mia macchina" smette di esistere)
- separare le dipendenze di progetti diversi
- distribuire un'applicazione completa come una singola **immagine**
- scalare orizzontalmente senza dover toccare il sistema operativo host

I tre concetti fondamentali sono:

1. **Immagine (image)**: pacchetto immutabile e versionato che contiene OS minimo + applicazione + dipendenze.
2. **Container**: istanza in esecuzione di un'immagine.
3. **Volume / Rete**: meccanismi per persistere i dati e far comunicare i container tra loro o con l'host.

Su Windows i container Linux non girano nativamente: girano dentro una piccola VM Linux gestita da WSL2. Capire questo dettaglio è importante perché spiega molti comportamenti (filesystem, rete, performance) che vedremo più avanti.

---

## 2. Prerequisiti e verifiche iniziali

Prima di installare qualsiasi cosa, verifica che il sistema sia pronto.

### 2.1 Versione di Windows

Per funzionare bene con Docker e WSL2 è consigliato:

- Windows 11 (qualsiasi build recente), oppure
- Windows 10 22H2 con build >= 19045

Da PowerShell:

```powershell
winver
```

### 2.2 Virtualizzazione hardware

In `Task Manager → Performance → CPU` deve comparire `Virtualization: Enabled`. Se è disabilitata, abilitala dal BIOS/UEFI (Intel VT-x o AMD-V / SVM Mode).

### 2.3 Funzionalità Windows necessarie

Da **PowerShell come amministratore**:

```powershell
Get-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform
Get-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
```

Se compaiono come `Disabled`, abilitale:

```powershell
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```

Riavvia il PC se hai modificato qualcosa.

### 2.4 WSL2 aggiornato e Ubuntu in versione 2

```powershell
wsl --version
wsl --status
wsl -l -v
```

Devi vedere:

- una versione di WSL recente (idealmente >= 2.0)
- la versione predefinita impostata a **2**
- la tua distribuzione Ubuntu in stato `Running` o `Stopped` con **VERSION 2**

Se la tua Ubuntu è in versione 1, convertila e aggiorna WSL:

```powershell
wsl --set-default-version 2
wsl --set-version Ubuntu 2
wsl --update
wsl --shutdown
```

### 2.5 Verifica della firma digitale degli installer (riproducibile)

Quando scaricherai Docker Desktop, **non limitarti al click destro → Proprietà**. Controlla la firma da PowerShell, è il modo riproducibile e non bypassabile:

```powershell
Get-AuthenticodeSignature 'C:\Users\<tuo-utente>\Downloads\Docker Desktop Installer.exe' |
  Format-List Status, SignerCertificate, TimeStamperCertificate
```

`Status` deve essere `Valid` e il `SignerCertificate.Subject` deve contenere `Docker Inc.`.

---

## 3. Configurazione di WSL2 e di Windows

Queste configurazioni vanno fatte **prima** di installare Docker, perché influenzano molto la qualità dell'esperienza.

### 3.1 `.wslconfig`: limiti globali della VM WSL2

Crea o modifica il file `C:\Users\<tuo-utente>\.wslconfig` con Blocco note. Valori sensati per una macchina con 16 GB RAM / 8 core:

```ini
[wsl2]
memory=6GB
processors=4
swap=2GB
localhostForwarding=true
# Nuovo modello di rete (Windows 11 22H2+/23H2+).
# Espone le porte dei container direttamente sull'host, senza port forwarding NAT.
# Risolve molti problemi di accesso ai container dal browser di Windows e da altre macchine.
networkingMode=mirrored
# Le opzioni seguenti vanno dentro [experimental] solo se networkingMode=mirrored è attivo
[experimental]
hostAddressLoopback=true
autoMemoryReclaim=gradual
sparseVhd=true
```

Note importanti:

- `networkingMode=mirrored` è disponibile su Windows 11 22H2 e successivi e cambia radicalmente il modello di rete: i container espongono le porte come se girassero su Windows nativo. Se la tua versione non lo supporta, **commenta la riga** (precedi con `#`).
- `autoMemoryReclaim=gradual` recupera la RAM non più usata dalla VM WSL: senza, la VM tende a tenere occupata tutta la `memory=` impostata.
- `sparseVhd=true` permette al disco virtuale di restituire spazio a Windows quando i file vengono cancellati (in passato bisognava farlo manualmente con `compact vdisk`).

Dopo aver salvato il file, da PowerShell:

```powershell
wsl --shutdown
```

E riapri Ubuntu.

### 3.2 Configurazione di Ubuntu (`/etc/wsl.conf`)

Dentro Ubuntu:

```bash
sudo nano /etc/wsl.conf
```

Contenuto consigliato:

```ini
[boot]
systemd=true

[network]
generateResolvConf=true
# Se sei dietro VPN aziendali e hai problemi di DNS,
# disabilita generateResolvConf=true e gestisci /etc/resolv.conf manualmente.

[interop]
enabled=true
appendWindowsPath=true

[user]
default=<tuo-utente-ubuntu>
```

`systemd=true` è **necessario** se userai Docker Engine nativo (Opzione B) per avviare il demone come servizio.

Da PowerShell:

```powershell
wsl --shutdown
```

Riapri Ubuntu e verifica:

```bash
ps -p 1 -o comm=
```

Deve restituire `systemd`.

### 3.3 Line endings (CRLF vs LF) — il problema #1 di chi sviluppa su Windows

È la singola fonte di errori più frequente per chi lavora con Docker su WSL2: file `.sh`, `Dockerfile` o entrypoint salvati con line ending Windows (CRLF) che esplodono dentro un container Linux con messaggi tipo `bash\r: No such file or directory`.

Configura **una volta per tutte**:

```bash
git config --global core.autocrlf input
git config --global core.eol lf
```

E nel root di **ogni** progetto crea un `.gitattributes`:

```gitattributes
* text=auto eol=lf

# Eccezioni per file che devono restare CRLF (script Windows)
*.bat   text eol=crlf
*.cmd   text eol=crlf
*.ps1   text eol=crlf

# Binari
*.png   binary
*.jpg   binary
*.pdf   binary
```

Se hai già file con CRLF in repo:

```bash
git add --renormalize .
git commit -m "Normalize line endings to LF"
```

### 3.4 Time skew di WSL2

Dopo che il PC va in sleep, l'orologio della VM WSL2 può sfasarsi rispetto all'host. Sintomo tipico: i `docker pull` falliscono per certificati TLS "non ancora validi". Soluzione una tantum:

```bash
sudo hwclock -s
```

Soluzione permanente:

```bash
sudo apt-get install -y systemd-timesyncd
sudo systemctl enable --now systemd-timesyncd
```

---

## 4. Quale Docker installare? Due strade possibili

Hai due opzioni, entrambe valide. Sceglierne una con cognizione di causa è già da professionista.

### Opzione A — Docker Desktop (consigliata se sei agli inizi)

- Installer ufficiale per Windows
- Integrazione automatica con WSL2 (espone il socket Docker dentro Ubuntu)
- GUI per immagini, container, volumi, Kubernetes locale
- Include `docker scout` per la scansione CVE
- **Licenza**: gratuita per uso personale, educational, open source e piccole aziende. La soglia attuale (verifica i termini correnti su `docker.com`) richiede licenza a pagamento per aziende con **più di 250 dipendenti** *o* **più di 10 M$ di fatturato annuo**.

### Opzione B — Docker Engine direttamente dentro WSL2 Ubuntu (consigliata se vuoi controllo totale o evitare la licenza Desktop)

- Nessuna GUI, nessuna licenza commerciale
- Identico a un server Linux di produzione (massima fedeltà)
- Devi gestire tu l'avvio del demone (systemd in WSL2)

In questa guida configureremo entrambe; **scegli quella che preferisci** e segui solo la sezione relativa.

---

## 5. Opzione A — Installazione Docker Desktop con backend WSL2

### 5.1 Download

Scarica l'installer ufficiale **solo** da `https://www.docker.com/products/docker-desktop/`.

Verifica la firma con `Get-AuthenticodeSignature` (vedi sezione 2.5). Se `Status` non è `Valid`, **non eseguirlo**.

### 5.2 Installazione

1. Esegui l'installer come amministratore.
2. Lascia **selezionata** la voce **Use WSL 2 instead of Hyper-V** (è il default).
3. Termina l'installazione e riavvia se richiesto.

### 5.3 Configurazione iniziale di Docker Desktop

Avvia Docker Desktop. Al primo avvio:

1. Accetta i termini di servizio dopo averli letti.
2. Salta il login con Docker Hub se non ti serve subito.
3. Apri **Settings (⚙️)** e configura:

**Settings → General**

- Spunta **Start Docker Desktop when you sign in to your computer** se vuoi avvio automatico.
- Verifica che **Use the WSL 2 based engine** sia attivo.
- **NON** spuntare *Expose daemon on tcp://localhost:2375 without TLS*: è un'esposizione del demone senza autenticazione, equivalente a dare root all'intera macchina.

**Settings → Resources → WSL Integration**

- Attiva **Enable integration with my default WSL distro**.
- Sotto **Enable integration with additional distros** attiva la tua **Ubuntu**.
- Clicca **Apply & Restart**.

**Settings → Resources → Advanced**

Con backend WSL2 questi valori (CPU, RAM, swap) **vengono scritti nel `.wslconfig` che hai già preparato** nella sezione 3.1. La GUI e il file `.wslconfig` configurano la stessa VM. Se hai già un `.wslconfig` corretto, puoi lasciare la GUI in pace.

**Settings → Docker Engine**

Questo pannello è il `daemon.json`. Sostituisci il contenuto con la configurazione professionale della **sezione 7**.

### 5.4 Verifica dell'integrazione WSL2

Apri il terminale **Ubuntu** (non PowerShell) ed esegui:

```bash
docker version
docker info
docker run --rm hello-world
```

Devi vedere il messaggio `Hello from Docker!`.

---

## 6. Opzione B — Docker Engine nativo dentro WSL2 Ubuntu

### 6.1 Pulizia di eventuali installazioni precedenti

```bash
sudo apt-get remove -y docker docker-engine docker.io containerd runc
```

### 6.2 Aggiunta del repository ufficiale Docker

```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg lsb-release

sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
    sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo $VERSION_CODENAME) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### 6.3 Installazione di Docker Engine, CLI, Compose e plugin

```bash
sudo apt-get update
sudo apt-get install -y \
  docker-ce \
  docker-ce-cli \
  containerd.io \
  docker-buildx-plugin \
  docker-compose-plugin
```

### 6.4 Avvio e abilitazione del demone Docker

(systemd dev'essere già attivo per quanto fatto in 3.2)

```bash
sudo systemctl enable --now docker
sudo systemctl status docker
```

---

## 7. Configurazione professionale del daemon (`daemon.json`)

Questo è il file che separa un'installazione "casalinga" da una professionale. Si trova in:

- **Docker Desktop**: `Settings → Docker Engine` (lo modifichi dalla GUI)
- **Docker Engine in WSL2**: `/etc/docker/daemon.json` (crealo se non esiste)

Configurazione consigliata:

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "5",
    "compress": "true"
  },
  "default-address-pools": [
    { "base": "172.30.0.0/16", "size": 24 }
  ],
  "live-restore": true,
  "userland-proxy": false,
  "no-new-privileges": true,
  "default-ulimits": {
    "nofile": { "Hard": 65536, "Soft": 65536 }
  },
  "features": {
    "buildkit": true,
    "containerd-snapshotter": true
  },
  "experimental": false
}
```

Cosa fa ogni opzione:

- **log-driver / log-opts**: ruota i log dei container (evita di riempire il disco). 10 MB × 5 file compressi = ~50 MB max per container. Il daemon accetta `compress` come stringa `"true"` perché internamente lo parsa lui — se vuoi essere ortodosso puoi anche scrivere `true` senza virgolette. Lascia `"true"` se preferisci uniformità con `max-size` e `max-file`.
- **default-address-pools**: riserva un range di IP privati alle reti Docker per evitare conflitti con la rete aziendale o VPN.
- **live-restore**: i container restano attivi anche se riavvii il demone.
- **userland-proxy: false**: usa `iptables` direttamente, più performante. **Avvertenza**: su alcune build di Docker Desktop con backend WSL2 questo può rendere temporaneamente non raggiungibili le porte da Windows finché non riavvii il container. Se noti regressioni, riportalo a `true`.
- **no-new-privileges**: i processi nel container non possono ottenere privilegi extra (anti-escalation).
- **default-ulimits**: alza i file descriptor (utile per server web, DB).
- **buildkit**: abilita il builder moderno (cache migliore, build parallele, secrets).
- **containerd-snapshotter**: abilita lo storage driver basato su containerd: necessario per supportare immagini multi-piattaforma e SBOM/provenance generati da `buildx` direttamente nello store locale.

> Nota: la guida originale aveva anche `"icc": false`. È una buona pratica di hardening, ma **disabilita la comunicazione tra container sulla bridge di default**. Se la attivi, devi creare reti user-defined per ogni progetto (Compose lo fa automaticamente, `docker run` semplici no). Per chi inizia ho preferito lasciarla fuori dal default per evitare confusione: aggiungila quando hai dimestichezza.

Dopo aver salvato:

- Su Docker Desktop: **Apply & Restart**.
- Su Docker Engine: `sudo systemctl restart docker`.

Verifica:

```bash
docker info | grep -E "Logging Driver|Live Restore|Default Address|userland|driver-type"
```

---

## 8. Permessi: usare Docker senza `sudo`

Aggiungi il tuo utente al gruppo `docker`:

```bash
sudo groupadd docker 2>/dev/null || true
sudo usermod -aG docker $USER
```

Poi **chiudi del tutto il terminale** e riapri Ubuntu, oppure da PowerShell:

```powershell
wsl --shutdown
```

Il trucco di `newgrp docker` apre solo una sotto-shell con il nuovo gruppo: funziona ma genera confusione se poi apri altri terminali. Una `wsl --shutdown` è più pulita.

Verifica:

```bash
id -nG | tr ' ' '\n' | grep -x docker
docker run --rm hello-world
```

> **Avvertenza di sicurezza**: appartenere al gruppo `docker` equivale ad avere privilegi di root sull'host, perché un container può montare `/`. Su una macchina condivisa o in produzione, valuta **rootless mode** (sezione 21).

---

## 9. Verifica finale dell'installazione

Controlli che farebbe un professionista subito dopo l'installazione:

```bash
docker version                 # client + server entrambi visibili
docker info                    # nessun WARNING grave
docker compose version         # Compose v2 disponibile
docker buildx version          # builder moderno disponibile
docker run --rm hello-world    # container di test
docker network ls              # bridge, host, none presenti
docker system df               # spazio occupato
```

Su WSL2 moderno (kernel 5.15+), Docker usa **cgroup v2** ed è il default. Se `docker info` mostra warning specifici su cgroup v1, di solito è perché `/etc/wsl.conf` o un kernel custom hanno forzato v1: in tal caso, rimuovi il forzamento.

---

## 10. Performance: dove tenere i progetti

Questo è il singolo cambiamento che dà l'impatto più grande sulla velocità.

- **MAI** sviluppare con bind mount da `/mnt/c/...`. Il filesystem 9P che WSL2 usa per accedere a NTFS è ordini di grandezza più lento per operazioni a piccoli file (`npm install`, `pip install`, `composer install`...).
- Tieni i progetti **dentro il filesystem WSL2**, ad esempio `~/progetti/` (in Ubuntu) oppure `\\wsl.localhost\Ubuntu\home\<utente>\progetti\` (visto da Windows Explorer).
- Apri VS Code con il comando `code .` da dentro la cartella WSL: VS Code si avvia in modalità *Remote - WSL* e tutte le operazioni sui file vanno alla velocità nativa di ext4.

Se hai già un progetto su `C:\` e vuoi spostarlo:

```bash
mkdir -p ~/progetti
cp -a /mnt/c/Users/<utente>/<progetto> ~/progetti/
cd ~/progetti/<progetto>
```

---

## 11. Struttura di progetto consigliata

```
mio-progetto/
├── .devcontainer/
│   └── devcontainer.json
├── .dockerignore
├── .gitattributes
├── .env.example
├── Dockerfile
├── docker-compose.yml
├── docker-compose.override.yml
├── docker-compose.prod.yml
├── compose.lint.yml
└── src/
```

### `.dockerignore` (sempre, fin dal primo giorno)

Riduce drasticamente il **build context** e impedisce di includere file sensibili nelle immagini:

```
.git
.gitignore
.gitattributes
node_modules
__pycache__
*.pyc
.env
.env.*
!.env.example
.vscode
.idea
*.log
dist
build
coverage
.DS_Store
Thumbs.db
.dockerignore
Dockerfile*
docker-compose*
README.md
.devcontainer
```

---

## 12. `Dockerfile` professionale

Esempio per un'app Node.js, ma il pattern (multi-stage, non-root, healthcheck, base image pinnata, cache mount, BuildKit secrets) vale per qualsiasi linguaggio.

```dockerfile
# syntax=docker/dockerfile:1.7

# ---------- Stage 1: build ----------
FROM node:20.14-alpine AS build
WORKDIR /app

COPY package*.json ./
RUN --mount=type=cache,target=/root/.npm npm ci

COPY . .
RUN npm run build

# ---------- Stage 2: runtime ----------
FROM node:20.14-alpine AS runtime
ENV NODE_ENV=production

# curl per healthcheck (più portabile di wget BusyBox).
RUN apk add --no-cache curl tini

WORKDIR /app

# Utente non-root
RUN addgroup -S app && adduser -S app -G app

COPY --from=build /app/package*.json ./
RUN --mount=type=cache,target=/root/.npm npm ci --omit=dev && npm cache clean --force
COPY --from=build /app/dist ./dist

USER app
EXPOSE 3000

# tini gestisce i segnali e fa reaping degli zombie processes
ENTRYPOINT ["/sbin/tini", "--"]

HEALTHCHECK --interval=30s --timeout=5s --start-period=15s --retries=3 \
  CMD curl -fsS http://localhost:3000/health || exit 1

CMD ["node", "dist/server.js"]
```

Punti professionali:

- `# syntax=...` abilita feature BuildKit moderne.
- **Multi-stage**: l'immagine finale non contiene il toolchain di build.
- **Cache mount** su npm: build più veloci e cache locale, non gonfia gli strati dell'immagine.
- **Versione major+minor pinnata** (`node:20.14-alpine`). Per produzione, considera anche il digest: `node:20.14-alpine@sha256:<hash>` (vedi sezione 20).
- **Utente non-root** (`USER app`).
- **`tini` come PID 1**: gestisce SIGTERM correttamente e fa reap dei figli.
- **HEALTHCHECK con `curl`**: più affidabile di `wget` di BusyBox, e dichiarativo.

### 12.1 Lint del Dockerfile con `hadolint`

Aggiungi un servizio di lint nel tuo workflow:

```bash
docker run --rm -i hadolint/hadolint < Dockerfile
```

In CI, fallo bloccare il merge se ci sono regole di livello `error`. Le regole che valgono il loro peso in oro: `DL3008` (pin delle versioni di apt), `DL3018` (pin delle versioni di apk), `DL3025` (usa `JSON` form di `CMD`/`ENTRYPOINT`), `DL3059` (consolida `RUN`).

### 12.2 Distroless e immagini minimal

Quando l'app è in linguaggio compilato (Go, Rust) o quando non hai bisogno di shell, considera:

- `gcr.io/distroless/static-debian12` per binari Go statici
- `gcr.io/distroless/cc-debian12` per binari C/C++/Rust dinamici
- `scratch` per binari Go senza alcuna dipendenza

Riduce drasticamente la superficie d'attacco e il numero di CVE.

---

## 13. `docker-compose.yml` professionale

```yaml
name: mio-progetto

services:
  app:
    build:
      context: .
      target: runtime
    image: mio-progetto/app:${APP_VERSION:-dev}
    restart: unless-stopped
    env_file:
      - .env
    ports:
      - "127.0.0.1:3000:3000"
    networks:
      - backend
    depends_on:
      db:
        condition: service_healthy
    deploy:
      resources:
        limits:
          cpus: "1.0"
          memory: 512M
        reservations:
          memory: 128M
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
    read_only: true
    tmpfs:
      - /tmp
      - /var/run
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "5"
    # Hot-reload moderno (Compose v2.22+): sincronizza file e/o ricostruisce
    develop:
      watch:
        - action: sync
          path: ./src
          target: /app/src
          ignore:
            - node_modules/
        - action: rebuild
          path: package.json

  db:
    image: postgres:16.4-alpine
    restart: unless-stopped
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
      POSTGRES_DB: ${DB_NAME}
    secrets:
      - db_password
    volumes:
      - db_data:/var/lib/postgresql/data
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $${DB_USER} -d $${DB_NAME}"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  db_data:

networks:
  backend:
    driver: bridge

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

Pratiche da notare:

- **`name:`** sostituisce `COMPOSE_PROJECT_NAME`.
- **`restart: unless-stopped`** anziché `always` (rispetta lo stop manuale).
- **Bind sull'host limitato a `127.0.0.1`**: la porta non è esposta sulla LAN.
- **`read_only: true` + `tmpfs`**: filesystem di runtime immutabile.
- **`cap_drop: ALL` + `cap_add` mirato**: principio del minimo privilegio.
- **Limiti di risorse e reservations** espliciti.
- **Healthcheck del DB** con `depends_on: condition: service_healthy`.
- **Volume nominato** per i dati persistenti.
- **`secrets` invece di env vars** per password e chiavi (vedi sezione 15).
- **`develop.watch`** per hot-reload nativo di Compose, senza dover usare bind mount tradizionali.

Con `watch` attivo lo si avvia con:

```bash
docker compose up --watch
```

### 13.1 Profili: dev / test / prod nello stesso file

Compose v2 supporta i **profili**, un modo pulito per attivare servizi solo in certi ambienti.

```yaml
services:
  app:
    # ... come sopra
  
  mailhog:
    image: mailhog/mailhog:v1.0.1
    profiles: ["dev"]
    ports:
      - "127.0.0.1:8025:8025"

  loadtest:
    image: grafana/k6:0.50.0
    profiles: ["test"]
    volumes:
      - ./tests:/scripts
    command: run /scripts/load.js
```

Esecuzione:

```bash
docker compose --profile dev up        # app + db + mailhog
docker compose --profile test up       # app + db + loadtest
docker compose up                      # solo i servizi senza profilo
```

### 13.2 File multipli per ambienti diversi

Pattern standard:

- `docker-compose.yml`: definizione base, valida per tutti gli ambienti
- `docker-compose.override.yml`: caricato automaticamente, contiene override per **dev** (bind mount, debug port, ecc.)
- `docker-compose.prod.yml`: caricato esplicitamente in produzione

```bash
# Sviluppo (default)
docker compose up

# Produzione
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

---

## 14. Reti e volumi

### 14.1 Reti

- **bridge** (default): rete locale dell'host, NAT verso l'esterno.
- **host**: il container condivide lo stack di rete dell'host (utile per perf, rischioso).
- **none**: nessuna rete (per job isolati).
- **user-defined bridge**: la **devi** preferire alla bridge di default. Permette risoluzione DNS automatica per nome del servizio.

```bash
docker network create app-net
docker run -d --name api --network app-net mio/api
docker run -d --name web --network app-net mio/web   # può chiamare http://api
```

Compose crea automaticamente una user-defined bridge per ogni progetto.

### 14.2 Volumi

Tre tipi:

1. **Named volume** (gestito da Docker): `-v db_data:/var/lib/postgresql/data` → consigliato per dati persistenti.
2. **Bind mount** (cartella host): `-v $(pwd):/app` → utile in sviluppo. Su WSL2, **solo** se la cartella host è dentro il filesystem WSL (vedi sezione 10).
3. **tmpfs** (in RAM): `--tmpfs /tmp` → per dati effimeri.

Comandi utili:

```bash
docker volume ls
docker volume inspect db_data
docker volume prune          # rimuove i volumi NON usati (attenzione)
```

---

## 15. Gestione di variabili d'ambiente e secret

### 15.1 Regole base

- **Mai** committare il file `.env`. Committa solo `.env.example` con chiavi vuote o di esempio.
- Aggiungi `.env` e `.env.*` al `.gitignore` e al `.dockerignore`.
- In Compose, distingui:
  - `env_file:` → variabili visibili dentro il container (es. configurazione runtime)
  - file `.env` nella stessa cartella del `compose.yml` → variabili **per Compose stesso** (interpolation, es. `${APP_VERSION}`).

### 15.2 Secret in Compose (file)

Per password e chiavi private, usa `secrets:` (esempio già visto in sezione 13). I secret vengono montati in `/run/secrets/<nome>` e non finiscono nelle env, dove sarebbero leggibili da `docker inspect`.

### 15.3 Secret a build time (BuildKit)

Per token di build (es. `npm` registry privato, GitHub PAT) **mai** scriverli con `ARG`, finiscono negli strati dell'immagine. Usa BuildKit secrets:

```dockerfile
# syntax=docker/dockerfile:1.7
FROM node:20.14-alpine
WORKDIR /app
COPY package*.json ./
RUN --mount=type=secret,id=npmrc,target=/root/.npmrc \
    npm ci
```

Build:

```bash
DOCKER_BUILDKIT=1 docker build \
  --secret id=npmrc,src=$HOME/.npmrc \
  -t mia-app:dev .
```

### 15.4 In produzione

Usa un vault esterno: HashiCorp Vault, AWS Secrets Manager, Azure Key Vault, Doppler, 1Password Secrets Automation. **Mai** secret in chiaro su disco in produzione.

---

## 16. GPU passthrough NVIDIA (opzionale, per ML/AI)

Se hai una GPU NVIDIA e vuoi usarla dai container:

### 16.1 Prerequisiti

- Driver NVIDIA aggiornati su Windows (≥ 472.39 per WSL2).
- Verifica dentro Ubuntu:

```bash
nvidia-smi
```

Se il comando funziona dentro Ubuntu, il driver passa correttamente attraverso WSL2.

### 16.2 Installa NVIDIA Container Toolkit (in Ubuntu)

```bash
distribution=$(. /etc/os-release; echo $ID$VERSION_ID)
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | \
  sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

curl -s -L https://nvidia.github.io/libnvidia-container/$distribution/libnvidia-container.list | \
  sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
  sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit

# Configura il runtime di Docker
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```

### 16.3 Test

```bash
docker run --rm --gpus all nvidia/cuda:12.4.0-base-ubuntu22.04 nvidia-smi
```

In Compose:

```yaml
services:
  trainer:
    image: pytorch/pytorch:2.3.0-cuda12.1-cudnn8-runtime
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]
```

> Su Docker Desktop il GPU passthrough è abilitato automaticamente se il driver Windows è aggiornato; il toolkit non va installato manualmente.

---

## 17. Dev Containers con VS Code

I **Dev Containers** ti permettono di sviluppare *dentro* il container che andrà in produzione: stessa toolchain, stesse versioni, niente "funziona sulla mia macchina".

### 17.1 Estensione

Installa in VS Code: `Dev Containers` (di Microsoft) e `WSL`.

### 17.2 `.devcontainer/devcontainer.json`

```json
{
  "name": "mio-progetto",
  "dockerComposeFile": ["../docker-compose.yml", "../docker-compose.override.yml"],
  "service": "app",
  "workspaceFolder": "/app",
  "shutdownAction": "stopCompose",
  "remoteUser": "app",
  "features": {
    "ghcr.io/devcontainers/features/common-utils:2": {
      "installZsh": true,
      "configureZshAsDefaultShell": true
    },
    "ghcr.io/devcontainers/features/git:1": {}
  },
  "customizations": {
    "vscode": {
      "extensions": [
        "dbaeumer.vscode-eslint",
        "esbenp.prettier-vscode",
        "ms-azuretools.vscode-docker"
      ],
      "settings": {
        "editor.formatOnSave": true,
        "files.eol": "\n"
      }
    }
  },
  "forwardPorts": [3000],
  "postCreateCommand": "npm install"
}
```

Apri la cartella in VS Code, premi `F1` → **Dev Containers: Reopen in Container**.

---

## 18. `docker context`: gestire più ambienti

`docker context` ti permette di passare velocemente tra demoni Docker diversi (locale, server remoto via SSH, CI):

```bash
docker context ls
docker context create remote --docker "host=ssh://utente@server.example.com"
docker context use remote
docker ps                  # parla con il demone remoto
docker context use default # torna al locale
```

Utile per deploy "manuale" su un VPS o per debug di un ambiente di staging senza esporre il demone in TCP.

---

## 19. Manutenzione: comandi quotidiani

```bash
# Vedere cosa sta girando
docker ps
docker ps -a                    # anche quelli stoppati

# Loggare un container
docker logs -f --tail=100 <container>

# Entrare dentro un container
docker exec -it <container> sh   # o bash, dipende dall'immagine

# Vedere risorse usate dai container
docker stats

# Pulizia spazio disco (lo farai spesso)
docker system df                 # quanto occupa?
docker image prune               # rimuove immagini "dangling"
docker container prune           # rimuove container stoppati
docker volume prune              # rimuove volumi orfani (CAUTELA)
docker builder prune             # cache di build
docker system prune -a --volumes # pulizia totale (DISTRUTTIVO)
```

### 19.1 Recupero spazio del VHDX (se non hai abilitato `sparseVhd`)

Su WSL2 il file VHDX cresce e (senza `sparseVhd=true`) non restituisce spazio a Windows. Per farlo manualmente, da PowerShell amministratore, dopo aver fatto `docker system prune`:

```powershell
wsl --shutdown

# Il path dipende da come hai installato Docker:
# - Docker Desktop recente: lo storage è dentro la distro WSL (vedi sotto)
# - Docker Desktop legacy: %LOCALAPPDATA%\Docker\wsl\disk\docker_data.vhdx
# - Docker Engine in Ubuntu: il disco è quello di Ubuntu stessa
#   (%LOCALAPPDATA%\Packages\CanonicalGroupLimited.Ubuntu*\LocalState\ext4.vhdx)

# Trova i tuoi VHDX:
Get-ChildItem -Recurse -Filter "*.vhdx" "$env:LOCALAPPDATA"

# Compatta uno specifico VHDX:
diskpart
# dentro diskpart:
select vdisk file="C:\percorso\al\tuo\file.vhdx"
attach vdisk readonly
compact vdisk
detach vdisk
exit
```

Se hai abilitato `sparseVhd=true` in `.wslconfig` (sezione 3.1), questo passaggio è **automatico** dopo qualche tempo.

---

## 20. Sicurezza: hardening, SBOM, provenance, scansione

Un professionista applica almeno questi controlli.

### 20.1 Regole di base

1. **Non eseguire mai container come root** se non strettamente necessario (`USER app` nel Dockerfile).
2. **Usa immagini ufficiali** o verificate (Docker Official, Verified Publisher).
3. **Pin delle versioni**: mai `:latest` in produzione. Usa il tag major+minor (`postgres:16.4`) e, idealmente, il **digest**.
4. **Filesystem read-only** quando l'app non deve scrivere (`read_only: true`).
5. **Drop di tutte le capability** e re-aggiunta solo di quelle necessarie (`cap_drop: ALL`, `cap_add: NET_BIND_SERVICE`).
6. **Niente `--privileged`** se non strettamente indispensabile.
7. **Mai esporre il socket** Docker (`/var/run/docker.sock`) dentro un container, salvo casi consapevoli (CI/CD): equivale a dare root all'host.
8. **Limita le porte** con bind esplicito a `127.0.0.1` se il servizio non deve essere raggiungibile dalla rete.
9. **Aggiorna regolarmente**: `docker pull` periodico delle base image, ricostruzione settimanale delle immagini.

### 20.2 Pin con digest (riproducibilità reale)

```dockerfile
FROM node:20.14-alpine@sha256:7c3b2... AS build
```

Il digest è immutabile: anche se domani il tag `20.14-alpine` viene ripubblicato con CVE diverse, la tua build pesca esattamente lo stesso layer. Strumenti come **Renovate** o **Dependabot** automatizzano l'aggiornamento dei digest pin.

### 20.3 Scansione CVE

```bash
# Docker Scout (richiede login a Docker Hub)
docker scout cves mio-progetto/app:1.0.0
docker scout recommendations mio-progetto/app:1.0.0

# Trivy (open source, ottimo per CI)
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy:latest image mio-progetto/app:1.0.0

# Grype
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  anchore/grype:latest mio-progetto/app:1.0.0
```

In CI, fai fallire la pipeline su CVE di livello `HIGH`/`CRITICAL` con fix disponibili.

### 20.4 SBOM (Software Bill of Materials) e provenance

Generare l'SBOM diventa requisito di compliance (NIST SSDF, EU Cyber Resilience Act). BuildKit lo fa nativamente:

```bash
docker buildx build \
  --sbom=true \
  --provenance=mode=max \
  -t mio-progetto/app:1.0.0 \
  --push .
```

Per generare un SBOM standalone con Syft:

```bash
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  anchore/syft:latest mio-progetto/app:1.0.0 -o spdx-json > sbom.spdx.json
```

### 20.5 Firma delle immagini (Cosign)

Se le pubblichi su un registry, firmale:

```bash
cosign generate-key-pair
cosign sign --key cosign.key registry.example.com/mio-progetto/app:1.0.0
cosign verify --key cosign.pub registry.example.com/mio-progetto/app:1.0.0
```

Cosign supporta anche firma keyless via OIDC (GitHub Actions, GitLab CI), che è il pattern moderno.

### 20.6 Linting di Compose

```bash
docker compose config            # valida il file e mostra il merge finale
docker compose config --services # elenca solo i servizi
```

E in CI:

```bash
docker run --rm -v "$PWD":/app -w /app \
  ghcr.io/zegl/kube-score:latest score docker-compose.yml || true
```

---

## 21. Rootless mode (avanzato, opzionale)

Se vuoi eliminare anche il rischio del gruppo `docker`, installa Docker in **rootless mode**. Dentro Ubuntu:

```bash
sudo apt-get install -y uidmap dbus-user-session slirp4netns fuse-overlayfs
curl -fsSL https://get.docker.com/rootless | sh

# Aggiungi al tuo ~/.bashrc o ~/.zshrc:
echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
echo 'export DOCKER_HOST=unix:///run/user/$(id -u)/docker.sock' >> ~/.bashrc
source ~/.bashrc

systemctl --user enable --now docker
```

In rootless mode hai qualche limitazione (porte < 1024 richiedono `setcap`, alcune feature di rete, niente `--net=host` cross-namespace) ma una superficie d'attacco molto più ridotta.

---

## 22. Aggiornamenti

### 22.1 Docker Desktop

- L'app notifica gli aggiornamenti. Aggiorna **almeno una volta al mese** per ricevere patch di sicurezza.
- Prima di un major update, fai `docker system df` e considera un backup dei volumi importanti (sezione 23).

### 22.2 Docker Engine in Ubuntu

```bash
sudo apt-get update
sudo apt-get upgrade -y \
  docker-ce docker-ce-cli containerd.io \
  docker-buildx-plugin docker-compose-plugin
```

### 22.3 Aggiornamento delle base image

```bash
# Lista tutte le immagini e fa pull dell'ultima del tag
docker image ls --format '{{.Repository}}:{{.Tag}}' | grep -v '<none>' | xargs -L1 docker pull

# Ricostruisci le immagini dei tuoi progetti
docker compose build --pull --no-cache
```

In CI puoi automatizzare questo con un cron settimanale.

---

## 23. Backup dei dati persistenti

```bash
# Backup di un volume nominato
docker run --rm \
  -v db_data:/data:ro \
  -v "$(pwd)":/backup \
  alpine tar czf /backup/db_data_$(date +%F).tar.gz -C /data .

# Restore
docker run --rm \
  -v db_data:/data \
  -v "$(pwd)":/backup \
  alpine sh -c "cd /data && tar xzf /backup/db_data_2026-05-07.tar.gz"
```

### 23.1 Backup logico per database

Per i database, **preferisci sempre** un dump logico al backup del volume a freddo:

```bash
# PostgreSQL
docker exec -t mio-progetto-db-1 \
  pg_dump -U $DB_USER -d $DB_NAME --format=custom \
  > backup_$(date +%F).dump

# Restore
docker exec -i mio-progetto-db-1 \
  pg_restore -U $DB_USER -d $DB_NAME --clean --if-exists \
  < backup_2026-05-07.dump
```

### 23.2 Backup del `.wslconfig` e dell'intera distro

Per non rifare tutto se cambi PC:

```powershell
# Esporta l'intera distro Ubuntu
wsl --export Ubuntu D:\backup\ubuntu-2026-05-07.tar

# Su un'altra macchina:
wsl --import Ubuntu C:\WSL\Ubuntu D:\backup\ubuntu-2026-05-07.tar --version 2
```

E copia anche `C:\Users\<utente>\.wslconfig` e `~/.docker/config.json`.

---

## 24. Logging centralizzato

In sviluppo va bene `json-file`. In produzione vorrai mandare i log a un sistema centralizzato. Scelte tipiche:

- **journald**: drop-in su qualsiasi host con systemd, integrato con `journalctl`.
- **fluentd / fluent-bit**: standard de facto per pipeline log.
- **gelf**: per Graylog.
- **awslogs / gcplogs**: per CloudWatch / Cloud Logging.
- **Loki** (via driver `loki`, esterno): pairing tipico con Grafana.

Esempio per journald:

```json
{
  "log-driver": "journald",
  "log-opts": {
    "tag": "{{.Name}}/{{.ID}}"
  }
}
```

Per servizio in Compose:

```yaml
services:
  app:
    logging:
      driver: fluentd
      options:
        fluentd-address: "tcp://logs.example.com:24224"
        tag: "myproj.app"
```

Considera che alcuni driver sono **bloccanti**: se il backend è giù, il container può rallentare o bloccarsi sul `stdout`. Per ambienti di produzione, valuta `mode: non-blocking` con buffer adeguato:

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "mode": "non-blocking",
    "max-buffer-size": "4m",
    "max-size": "10m",
    "max-file": "5"
  }
}
```

---

## 25. Troubleshooting rapido

| Sintomo | Causa probabile | Soluzione |
|---|---|---|
| `docker: command not found` in Ubuntu | Integrazione WSL non attiva o PATH non rinfrescato | Settings → Resources → WSL Integration; riapri il terminale |
| `permission denied` su `/var/run/docker.sock` | Utente non nel gruppo `docker` | `sudo usermod -aG docker $USER` + `wsl --shutdown` |
| `Cannot connect to the Docker daemon` | Demone non avviato | `sudo systemctl start docker` o avvia Docker Desktop |
| Container lentissimi su bind mount Windows | Stai montando da `/mnt/c/...` | Sposta il progetto dentro il filesystem WSL (`~/progetti/...`) |
| Disco Windows si riempie | VHDX di WSL/Docker cresciuto | Abilita `sparseVhd=true`, oppure compatta a mano (sezione 19.1) |
| `pull access denied` | Immagine privata senza login | `docker login` |
| Build sempre da capo | Cache invalidata da `COPY .` prima di `npm ci` | Copia prima `package*.json`, poi installa, poi copia il resto |
| `bash\r: No such file or directory` | Script con line ending CRLF | Vedi sezione 3.3 (`.gitattributes` + `git add --renormalize .`) |
| `x509: certificate has expired or is not yet valid` su `docker pull` | Orologio di WSL2 sfasato | `sudo hwclock -s` o installa `systemd-timesyncd` (sezione 3.4) |
| Le porte dei container non si vedono da Windows | Modalità di rete WSL2 non ottimale | Prova `networkingMode=mirrored` in `.wslconfig` (sezione 3.1) |
| DNS che non risolve dentro i container in azienda | VPN/proxy aziendale | `--dns=8.8.8.8` su `docker run` o `dns:` in compose; oppure `dns` in `daemon.json` |
| `docker compose up` fallisce con "no such service" su un profilo | Manca `--profile <nome>` | `docker compose --profile dev up` |
| `failed to register layer ... no space left on device` | Volume di Docker pieno | `docker system prune -af --volumes` poi `compact vdisk` |
| `iptables: No chain/target/match by that name` | Firewall in WSL2 obsoleto | `sudo update-alternatives --set iptables /usr/sbin/iptables-legacy` |
| Container che reagisce male a Ctrl+C | Manca init/PID 1 sano | Aggiungi `tini` o `init: true` nel compose |

---

## 26. Prossimi passi consigliati

Una volta padroneggiate le basi, approfondisci in quest'ordine:

1. **BuildKit avanzato**: cache mount remote (`type=registry`), build multi-piattaforma con `buildx create --use --platform linux/amd64,linux/arm64`.
2. **Registry privato** (Harbor, GitHub Container Registry, GitLab Container Registry) con CI/CD (GitHub Actions, GitLab CI) che builda, firma con Cosign, scansiona con Trivy e pubblica.
3. **Compose Profiles e `develop.watch`** in profondità, e migrazione dei "vecchi" `docker-compose.dev.yml` ai profili.
4. **Orchestrazione**: Docker Swarm per iniziare (Compose-compatible), poi Kubernetes (k3d/kind in locale) quando ti servirà davvero.
5. **Osservabilità**: Prometheus + Grafana + Loki + cAdvisor per metriche e log dei container.
6. **Policy as code**: scansione automatica con Trivy/Scout in pipeline, blocco di immagini con CVE critiche, controllo SBOM con Open Policy Agent / Conftest.
7. **Supply chain security**: Cosign keyless via OIDC, attestation in-toto, Sigstore.

---

## 27. Checklist del professionista

Considera l'installazione completata solo quando puoi spuntare **tutte** queste voci:

### Sistema

- [ ] Virtualizzazione attiva nel BIOS/UEFI
- [ ] WSL2 aggiornato (`wsl --update`), Ubuntu in versione 2, systemd attivo
- [ ] `.wslconfig` configurato con limiti RAM/CPU sensati e (se supportato) `networkingMode=mirrored`, `sparseVhd=true`, `autoMemoryReclaim=gradual`
- [ ] `git config core.autocrlf input` e `.gitattributes` con `eol=lf` di default

### Docker

- [ ] Docker Desktop **oppure** Docker Engine installato e funzionante
- [ ] Integrazione WSL2 attiva sulla distro Ubuntu (se usi Desktop)
- [ ] `docker run --rm hello-world` funziona dentro Ubuntu **senza `sudo`**
- [ ] `daemon.json` configurato con log rotation, address pools, live-restore, no-new-privileges, BuildKit, containerd-snapshotter
- [ ] BuildKit, Compose v2 e Buildx disponibili (`docker buildx version`, `docker compose version`)

### Workflow

- [ ] Capisci la differenza tra immagine, container, volume e rete
- [ ] I tuoi progetti vivono dentro `~/progetti/` (filesystem WSL), **non** in `/mnt/c/...`
- [ ] Sai scrivere un `Dockerfile` multi-stage con utente non-root, `tini`/init, healthcheck, cache mount
- [ ] Lint di Dockerfile con `hadolint` integrato nel tuo workflow
- [ ] Sai scrivere un `docker-compose.yml` con limiti di risorse, healthcheck, reti dedicate, volumi nominati, secret, profiles e `develop.watch`
- [ ] Hai una `.devcontainer/devcontainer.json` per VS Code

### Sicurezza e operations

- [ ] Pin delle base image a major+minor (`node:20.14-alpine`), digest in produzione
- [ ] Scansione CVE in CI (Scout o Trivy) che blocca il merge su HIGH/CRITICAL con fix
- [ ] SBOM e provenance generati in build (`buildx --sbom --provenance=mode=max`)
- [ ] Workflow di pulizia periodica (`docker system df` / `prune`)
- [ ] Piano di backup per i volumi importanti (dump logico per i DB)
- [ ] Strategia di logging definita (json-file con rotazione in dev, driver centralizzato in prod)
- [ ] Conosci `docker context` per gestire ambienti remoti senza esporre il demone

Quando tutte le caselle sono spuntate, puoi considerarti operativo a livello professionale.
