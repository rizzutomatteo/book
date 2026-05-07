# Guida professionale all'installazione e configurazione di Docker su Windows + WSL2 Ubuntu

Questa guida è pensata per chi non ha **mai** usato Docker e vuole partire da zero, ottenendo un ambiente di lavoro **professionale, sicuro e pronto alla produzione**. È scritta per Windows 10/11 con WSL2 e Ubuntu già installato.

Ogni scelta tecnica è accompagnata da una spiegazione del **perché**: copiare comandi senza capire cosa fanno è il modo più rapido per avere un'installazione fragile.

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
- Strategia di gestione dei secret valida sia in sviluppo che in produzione
- Mitigazioni concrete per i rate limit di Docker Hub
- Pipeline CI con cache esterna BuildKit

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
15. Gestione di variabili d'ambiente e secret (dev e prod)
16. Rate limit di Docker Hub e come gestirli
17. BuildKit avanzato e cache CI
18. GPU passthrough NVIDIA (opzionale)
19. Dev Containers con VS Code
20. `docker context`: gestire più ambienti
21. Manutenzione: comandi quotidiani
22. Sicurezza: hardening, SBOM, provenance, scansione, firma
23. Rootless mode (avanzato, opzionale)
24. Tuning avanzato del daemon (opzionale)
25. Aggiornamenti
26. Backup dei dati persistenti
27. Logging centralizzato
28. Troubleshooting rapido
29. Prossimi passi consigliati
30. Checklist del professionista

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

**Perché questa distinzione conta:** un'immagine è come un eseguibile; un container è come un processo in esecuzione di quell'eseguibile. Puoi avere mille container dalla stessa immagine, ognuno isolato. Capirlo bene ti evita errori frequenti come "ho modificato il container e ora ho perso tutto al riavvio" (perché lo stato di un container è effimero per design — la persistenza vive nei volumi).

Su Windows i container Linux non girano nativamente: girano dentro una piccola VM Linux gestita da WSL2. Capire questo dettaglio è importante perché spiega molti comportamenti (filesystem, rete, performance) che vedremo più avanti.

---

## 2. Prerequisiti e verifiche iniziali

Prima di installare qualsiasi cosa, verifica che il sistema sia pronto. **Perché:** un'installazione Docker su un sistema mal configurato funziona spesso ma in modo subdolo (lento, fragile, con problemi di rete intermittenti). Spendere 10 minuti su questi controlli ti risparmia ore di debugging dopo.

### 2.1 Versione di Windows

Per funzionare bene con Docker e WSL2 è consigliato:

- Windows 11 (qualsiasi build recente), oppure
- Windows 10 22H2 con build >= 19045

Da PowerShell:

```powershell
winver
```

**Perché:** alcune funzionalità chiave di WSL2 (in particolare `networkingMode=mirrored` e `autoMemoryReclaim`) richiedono build moderne. Su build vecchie funziona ma con limiti notevoli.

### 2.2 Virtualizzazione hardware

In `Task Manager → Performance → CPU` deve comparire `Virtualization: Enabled`. Se è disabilitata, abilitala dal BIOS/UEFI (Intel VT-x o AMD-V / SVM Mode).

**Perché:** WSL2 usa una VM Hyper-V leggera, che richiede le estensioni di virtualizzazione hardware della CPU. Senza, WSL2 semplicemente non parte (o parte in modalità WSL1, che non supporta Docker nativamente).

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

**Perché WSL2 e non WSL1:** WSL1 è un layer di traduzione delle syscall Linux verso il kernel Windows. WSL2 è una vera VM con un kernel Linux completo. Docker richiede WSL2 perché ha bisogno di funzionalità del kernel Linux (cgroups, namespaces, overlayfs) che WSL1 non implementa.

### 2.5 Verifica della firma digitale degli installer (riproducibile)

Quando scaricherai Docker Desktop, **non limitarti al click destro → Proprietà**. Controlla la firma da PowerShell, è il modo riproducibile e non bypassabile:

```powershell
Get-AuthenticodeSignature 'C:\Users\<tuo-utente>\Downloads\Docker Desktop Installer.exe' |
  Format-List Status, SignerCertificate, TimeStamperCertificate
```

`Status` deve essere `Valid` e il `SignerCertificate.Subject` deve contenere `Docker Inc.`.

**Perché:** un installer firmato garantisce che il binario non è stato modificato dopo il rilascio. Il click destro mostra una GUI che alcuni malware sanno spoofare; il comando PowerShell fa una verifica crittografica vera e propria.

---

## 3. Configurazione di WSL2 e di Windows

Queste configurazioni vanno fatte **prima** di installare Docker, perché influenzano molto la qualità dell'esperienza. Modificarle dopo richiede riavvi e a volte ricostruzioni delle reti Docker.

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

# Richiesto per loopback affidabile in modalità mirrored.
hostAddressLoopback=true

# Recupera la RAM non più usata dalla VM (su WSL 2.0+ è in [wsl2], non più experimental).
autoMemoryReclaim=gradual

# Disco virtuale "sparse": restituisce spazio a Windows quando i file vengono cancellati.
sparseVhd=true
```

**Note importanti su versioni WSL e sezioni:**

- Su **WSL >= 2.0** (uscito a fine 2023) `autoMemoryReclaim`, `sparseVhd`, `networkingMode` e `hostAddressLoopback` vivono sotto `[wsl2]` come mostrato sopra.
- Su **WSL più vecchio** dovevano stare sotto una sezione `[experimental]`. Se `wsl --version` ti mostra una versione < 2.0, sposta queste quattro righe in:

  ```ini
  [experimental]
  networkingMode=mirrored
  hostAddressLoopback=true
  autoMemoryReclaim=gradual
  sparseVhd=true
  ```

  e in tal caso valuta anche di aggiornare WSL con `wsl --update`.

- `networkingMode=mirrored` è disponibile da Windows 11 22H2 in poi e cambia radicalmente il modello di rete: i container espongono le porte come se girassero su Windows nativo. Se la tua versione non lo supporta o se devi usare VPN aziendali particolari, **commenta la riga** (precedi con `#`).

**Perché ogni opzione:**

- `memory` / `processors` / `swap`: senza questi valori WSL2 può prendere fino al 50% della tua RAM totale, lasciando il sistema affamato. Imposta valori conservativi rispetto a quanto ti serve davvero.
- `localhostForwarding=true`: senza questo, raggiungere `localhost:8080` da Windows quando il servizio gira in WSL non funziona in modalità non-mirrored.
- `networkingMode=mirrored`: il modello di rete legacy fa NAT su un'interfaccia virtuale. Funziona, ma genera mille piccoli problemi (porte non raggiungibili, IP che cambia ad ogni reboot, mDNS che non passa). `mirrored` fa vedere a WSL le interfacce di rete dell'host: meno fragile.
- `autoMemoryReclaim=gradual`: senza questo, una volta che WSL ha allocato 6 GB di RAM (vedi `memory=6GB`), li tiene fino al successivo `wsl --shutdown`. Con `gradual`, la VM rilascia gradualmente la memoria non usata.
- `sparseVhd=true`: storicamente il file VHDX di WSL2 cresceva ma non si riduceva mai. Con `sparseVhd` il rilascio è automatico (sebbene non immediato — può richiedere ore).

**Avvertenza VPN/firewall:** alcuni client VPN aziendali (in particolare Cisco AnyConnect, GlobalProtect) hanno problemi con `networkingMode=mirrored` perché aggiungono regole iptables che entrano in conflitto. Se dopo l'attivazione il DNS dentro WSL smette di funzionare quando sei in VPN, è la prima cosa da disattivare.

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

**Perché systemd:** senza systemd, in WSL2 non hai un init system completo. I servizi (Docker, PostgreSQL, etc.) non si avviano automaticamente al boot della distro e non hai `systemctl`. Con `systemd=true`, Ubuntu in WSL2 si comporta come un Ubuntu vero.

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

**Perché succede:** Linux interpreta letteralmente i caratteri di fine riga. Uno script che inizia con `#!/bin/bash\r\n` viene letto come "esegui l'interprete `/bin/bash\r`", che non esiste. L'errore è criptico perché l'`\r` è invisibile.

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

**Perché `.gitattributes` è meglio della config globale:** la config globale riguarda solo te. Il `.gitattributes` viaggia con il repo, quindi anche un collega con un setup diverso si ritroverà con line ending corretti.

### 3.4 Time skew di WSL2

Dopo che il PC va in sleep, l'orologio della VM WSL2 può sfasarsi rispetto all'host. Sintomo tipico: i `docker pull` falliscono per certificati TLS "non ancora validi" o "scaduti". Soluzione una tantum:

```bash
sudo hwclock -s
```

Soluzione permanente:

```bash
sudo apt-get install -y systemd-timesyncd
sudo systemctl enable --now systemd-timesyncd
```

**Perché succede:** la VM di WSL2 viene messa in pausa (non spenta) quando l'host va in sleep. Al risveglio, l'orologio interno della VM è rimasto fermo per ore o giorni. TLS richiede orologi sincronizzati per validare i certificati: un'ora sbagliata di 5 minuti basta a rompere tutto.

---

## 4. Quale Docker installare? Due strade possibili

Hai due opzioni, entrambe valide. Sceglierne una con cognizione di causa è già da professionista.

### Opzione A — Docker Desktop (consigliata se sei agli inizi)

- Installer ufficiale per Windows
- Integrazione automatica con WSL2 (espone il socket Docker dentro Ubuntu)
- GUI per immagini, container, volumi, Kubernetes locale
- Include `docker scout` per la scansione CVE
- **Licenza**: gratuita per uso personale, educational, open source e piccole aziende. La soglia attuale (verifica i termini correnti su `docker.com`) richiede licenza a pagamento per aziende con **più di 250 dipendenti** *o* **più di 10 M$ di fatturato annuo**.

**Perché sceglierla:** ti dà una GUI, gestisce per te l'aggiornamento del demone, ha il KubernetesDashboard incluso, mostra i log e i layer in modo visuale. Per imparare è più gentile.

### Opzione B — Docker Engine direttamente dentro WSL2 Ubuntu (consigliata se vuoi controllo totale o evitare la licenza Desktop)

- Nessuna GUI, nessuna licenza commerciale
- Identico a un server Linux di produzione (massima fedeltà)
- Devi gestire tu l'avvio del demone (systemd in WSL2)

**Perché sceglierla:** è il setup *vero* che troverai in produzione. Se domani devi fare debug di un problema su un server Linux, l'esperienza è identica al millimetro. Inoltre nessuna preoccupazione di licenza in azienda.

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
2. Salta il login con Docker Hub se non ti serve subito (ma vedi sezione 16 sui rate limit).
3. Apri **Settings (⚙️)** e configura:

**Settings → General**

- Spunta **Start Docker Desktop when you sign in to your computer** se vuoi avvio automatico.
- Verifica che **Use the WSL 2 based engine** sia attivo.
- **NON** spuntare *Expose daemon on tcp://localhost:2375 without TLS*: è un'esposizione del demone senza autenticazione, equivalente a dare root all'intera macchina. Qualsiasi processo locale (compresi script di pagine web maligne in browser) può controllare i tuoi container.

**Settings → Resources → WSL Integration**

- Attiva **Enable integration with my default WSL distro**.
- Sotto **Enable integration with additional distros** attiva la tua **Ubuntu**.
- Clicca **Apply & Restart**.

**Perché:** senza questo, il binario `docker` non sarà disponibile dentro Ubuntu. Devi attivarlo esplicitamente per ogni distro che vuoi che lo veda.

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
sudo apt-get remove -y \
  docker docker-engine docker.io docker-doc docker-compose \
  podman-docker containerd runc
```

**Perché tutti questi pacchetti:** `docker.io` è la versione storica nei repo Ubuntu (vecchia). `docker-compose` è la v1 standalone (deprecata, sostituita da `docker compose` plugin v2). `docker-doc` sono solo le man page del vecchio pacchetto. `podman-docker` è uno shim alternativo che può creare conflitti. Pulisci tutto per evitare che `apt` si confonda quando installi i pacchetti ufficiali Docker.

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

**Perché questa procedura:** `apt-key` (il vecchio modo di aggiungere chiavi GPG) è deprecato per problemi di sicurezza. Il nuovo standard è mettere la chiave in `/etc/apt/keyrings/` e referenziarla esplicitamente con `signed-by=` nel file `.list`. Così quella chiave firma solo il repo Docker, non altri.

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

**Perché `enable --now`:** `enable` lo registra per l'avvio automatico, `--now` lo avvia subito. È l'idioma standard di systemd per "voglio che parta sempre, e voglio che parta anche adesso".

---

## 7. Configurazione professionale del daemon (`daemon.json`)

Questo è il file che separa un'installazione "casalinga" da una professionale. Si trova in:

- **Docker Desktop**: `Settings → Docker Engine` (lo modifichi dalla GUI)
- **Docker Engine in WSL2**: `/etc/docker/daemon.json` (crealo se non esiste)

Configurazione consigliata come **default sicuro per principianti e intermedi**:

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
  "no-new-privileges": true,
  "default-ulimits": {
    "nofile": { "Hard": 65536, "Soft": 65536 }
  },
  "features": {
    "buildkit": true
  },
  "experimental": false
}
```

Spiegazione di ogni opzione:

- **`log-driver` / `log-opts`**: ruota i log dei container. Senza questo, un container loquace può riempire il disco in pochi giorni (i log JSON di Docker non hanno limite di default). 10 MB × 5 file compressi ≈ 50 MB max per container. Su `compress`: il daemon accetta sia la stringa `"true"` che il booleano `true`. Le versioni recenti preferiscono il booleano; lascialo come stringa solo se hai vincoli di tooling che fanno parsing strict del JSON con i tipi sbagliati.
- **`default-address-pools`**: riserva un range di IP privati (172.30.0.0/16) alle reti Docker. Senza questo, Docker pesca dal range 172.17.x e successivi, che spesso entra in conflitto con VPN aziendali (molte VPN usano 172.16/12). Cambia il range se anche 172.30 ti dà fastidio.
- **`live-restore: true`**: i container restano attivi anche se riavvii il demone Docker. Critico in produzione (restart del demone = downtime zero) e comodo in sviluppo (un crash di Docker Desktop non ti uccide tutti i container).
- **`no-new-privileges: true`**: i processi nel container non possono ottenere privilegi extra via `setuid`. Anti-escalation a livello di kernel.
- **`default-ulimits`**: alza i file descriptor (utile per server web, DB). Il default Linux di 1024 è ridicolmente basso per un server moderno.
- **`features.buildkit: true`**: abilita il builder moderno (cache migliore, build parallele, secrets, mount di tipo cache). Su Docker Desktop è già attivo di default; metterlo esplicito è un'assicurazione.
- **`experimental: false`**: dichiari esplicitamente che non vuoi feature sperimentali. Documenta l'intento.

### 7.1 Opzioni che NON ho messo nel default e perché

Tre opzioni che la versione precedente di questa guida includeva sono state spostate in **sezione 24 (Tuning avanzato)** perché possono causare regressioni se attivate alla cieca:

- **`userland-proxy: false`**: disabilita il proxy in user space e usa direttamente iptables. Più performante in teoria, ma su WSL2 ha causato regressioni di raggiungibilità delle porte da Windows in alcune build di Docker Desktop. Attivala solo se sai cosa stai facendo e dopo aver letto la sezione 24.
- **`containerd-snapshotter: true`**: cambia lo storage layer a quello di containerd (necessario per immagini multi-piattaforma e SBOM/provenance generati direttamente da `buildx`). Su un'installazione esistente, l'attivazione ti **invalida lo store delle immagini** (devono essere ri-pullate). Vale la pena, ma fallo consapevolmente: vedi sezione 24.
- **`icc: false`**: disabilita la inter-container communication sulla bridge di default. Ottimo per hardening, ma rompe `docker run` semplici tra container che si parlano. Aggiungila solo quando lavori sempre con reti user-defined (Compose lo fa per te, `docker run` no).

Dopo aver salvato:

- Su Docker Desktop: **Apply & Restart**.
- Su Docker Engine: `sudo systemctl restart docker`.

Verifica:

```bash
docker info | grep -E "Logging Driver|Live Restore|Default Address|Cgroup"
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

**Perché non usare `newgrp docker`:** apre solo una sotto-shell con il nuovo gruppo. Funziona ma genera confusione: in altri terminali aperti il gruppo non c'è ancora, e ti chiedi perché un comando funziona qui e non là. `wsl --shutdown` rinizializza pulito tutto.

Verifica:

```bash
id -nG | tr ' ' '\n' | grep -x docker
docker run --rm hello-world
```

> **Avvertenza di sicurezza importante:** appartenere al gruppo `docker` equivale ad **avere privilegi di root sull'host**, perché un container può montare `/` con `-v /:/host` e modificare qualsiasi file di sistema. Su una macchina condivisa, in produzione, o su un dev box con accesso a segreti aziendali, valuta seriamente **rootless mode** (sezione 23). Non è paranoia: è il modello di sicurezza documentato di Docker.

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

**Cosa cercare in `docker info`:**

- `Cgroup Driver: systemd` (su Ubuntu moderno è il default e quello giusto)
- `Cgroup Version: 2` (cgroup v2: kernel 5.8+ standard)
- `Storage Driver: overlay2` (o `overlayfs` con containerd-snapshotter)
- Nessun `WARNING:` rosso

Su WSL2 moderno (kernel 5.15+), Docker usa **cgroup v2** ed è il default. Se `docker info` mostra warning specifici su cgroup v1, di solito è perché `/etc/wsl.conf` o un kernel custom hanno forzato v1: in tal caso, rimuovi il forzamento.

---

## 10. Performance: dove tenere i progetti

Questo è il singolo cambiamento che dà l'impatto più grande sulla velocità.

- **MAI** sviluppare con bind mount da `/mnt/c/...`. Il filesystem 9P che WSL2 usa per accedere a NTFS è ordini di grandezza più lento per operazioni a piccoli file (`npm install`, `pip install`, `composer install`...).
- Tieni i progetti **dentro il filesystem WSL2**, ad esempio `~/progetti/` (in Ubuntu) oppure `\\wsl.localhost\Ubuntu\home\<utente>\progetti\` (visto da Windows Explorer).
- Apri VS Code con il comando `code .` da dentro la cartella WSL: VS Code si avvia in modalità *Remote - WSL* e tutte le operazioni sui file vanno alla velocità nativa di ext4.

**Perché è così lento `/mnt/c`:** WSL2 e Windows hanno filesystem completamente separati (la VM WSL ha il suo ext4 dentro un VHDX). Per accedere a NTFS dalla VM, c'è un protocollo di rete (9P) che fa proxying delle chiamate filesystem. Va bene per leggere occasionalmente un file, è disastroso per un `npm install` che fa decine di migliaia di stat/open/read piccolissimi.

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

**Perché è critico:** quando esegui `docker build .`, Docker invia *l'intera cartella corrente* al daemon come "build context". Senza `.dockerignore`, una `node_modules/` di 500 MB viene trasferita ad ogni build, anche se non finisce nell'immagine. Ancora peggio: un file `.env` con password viene letto da Docker, indicizzato nei layer di cache, e se mai pubblichi l'immagine quei segreti viaggiano con essa.

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

Punti professionali — e perché ognuno conta:

- `# syntax=docker/dockerfile:1.7`: abilita feature BuildKit moderne (cache mount, secrets, heredoc). Senza, alcune sintassi falliscono silenziosamente.
- **Multi-stage**: l'immagine finale non contiene il toolchain di build (compilatori, dev dependencies). Differenza tipica: 1.2 GB → 180 MB. Meno superficie d'attacco, pull più veloce.
- **Cache mount su npm**: build più veloci e cache locale, non gonfia gli strati dell'immagine. Senza, ogni `npm ci` riscarica tutto da internet ad ogni build.
- **Versione major+minor pinnata** (`node:20.14-alpine`). Per produzione, considera anche il digest: `node:20.14-alpine@sha256:<hash>` (vedi sezione 22). Il tag `latest` è una bomba a tempo: oggi ti dà Node 20, domani Node 22 con breaking changes.
- **`alpine`**: base minimale (~5 MB vs ~80 MB di Debian slim). Attenzione: Alpine usa musl libc invece di glibc, il che a volte rompe pacchetti compilati nativamente (es. alcuni binding di Node con `node-gyp`). In quei casi, ripiega su `node:20.14-slim` (Debian).
- **Utente non-root** (`USER app`): se l'app viene compromessa, l'attaccante ha i permessi di un utente normale dentro il container, non root. Cambia tutto in caso di kernel exploit.
- **`tini` come PID 1**: gestisce SIGTERM correttamente e fa reap dei figli. Senza, `docker stop` aspetta 10 secondi e poi `SIGKILL` brutale, e i processi figli diventano zombie.
- **Alternativa a `tini`**: in Compose (vedi sezione 13) puoi usare `init: true` come scorciatoia, che inietta un init minimale automaticamente. Comodo se non vuoi modificare il Dockerfile.
- **HEALTHCHECK con `curl`**: più affidabile di `wget` di BusyBox, e dichiarativo. Senza un healthcheck, Docker considera "running" qualunque processo che non sia crashato — anche un'app bloccata che non risponde a niente.

### 12.1 Lint del Dockerfile con `hadolint`

Aggiungi un servizio di lint nel tuo workflow:

```bash
docker run --rm -i hadolint/hadolint < Dockerfile
```

In CI, fallo bloccare il merge se ci sono regole di livello `error`. Le regole che valgono il loro peso in oro:

- `DL3008` (pin delle versioni di apt)
- `DL3018` (pin delle versioni di apk)
- `DL3025` (usa `JSON` form di `CMD`/`ENTRYPOINT`)
- `DL3059` (consolida `RUN`)

**Perché:** un `apt-get install -y curl` senza pin scarica la versione più nuova al momento della build. Build identica a 6 mesi di distanza? Diversa. Hai appena rotto la riproducibilità.

### 12.2 Distroless e immagini minimal

Quando l'app è in linguaggio compilato (Go, Rust) o quando non hai bisogno di shell, considera:

- `gcr.io/distroless/static-debian12` per binari Go statici
- `gcr.io/distroless/cc-debian12` per binari C/C++/Rust dinamici
- `scratch` per binari Go senza alcuna dipendenza

**Perché:** queste immagini non hanno `sh`, `apt`, `curl`, niente. Se un attaccante ottiene RCE nel tuo container, non ha strumenti per fare lateral movement. Differenza tra "il mio container è bucato" e "il mio container è bucato e l'attaccante sta scaricando tool in tempo reale". Inoltre meno CVE da patchare.

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
    init: true              # alternativa a tini se non vuoi toccare il Dockerfile
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

Pratiche da notare e perché:

- **`name:`** sostituisce `COMPOSE_PROJECT_NAME`. Esplicita il nome del progetto invece di derivarlo dalla cartella corrente: due cloni dello stesso repo in cartelle diverse non si pestano i piedi.
- **`restart: unless-stopped`** anziché `always`. Differenza: `always` riavvia anche dopo un `docker stop` manuale; `unless-stopped` rispetta lo stop manuale. È quasi sempre quello che vuoi.
- **`init: true`**: scorciatoia che inietta un init minimale (gestione segnali e reaping zombie) senza dover modificare il Dockerfile. Se hai già `tini` come ENTRYPOINT, è ridondante ma non dannoso.
- **Bind sull'host limitato a `127.0.0.1`**: la porta non è esposta sulla LAN. Senza questo, su `0.0.0.0:3000` il servizio è raggiungibile da chiunque sia sulla tua rete WiFi. Dimenticarlo è uno dei vettori più comuni di esposizione accidentale di servizi di sviluppo.
- **`read_only: true` + `tmpfs`**: filesystem di runtime immutabile. L'app può scrivere solo su `/tmp` (in RAM). Mitiga la stragrande maggioranza degli exploit di tipo "scrivi una webshell".
- **`cap_drop: ALL` + `cap_add` mirato**: principio del minimo privilegio. Di default un container ha decine di Linux capabilities (CHOWN, NET_RAW, ecc.) che servono raramente. Drop di tutto, riaggiungi solo ciò che serve.
- **Limiti di risorse e reservations** espliciti: senza, un container può saturare RAM e CPU dell'host. Con `reservations`, lo scheduler garantisce un minimo (utile in Compose con Swarm o k8s, neutro in Compose stand-alone ma documenta l'intento).
- **Healthcheck del DB** con `depends_on: condition: service_healthy`: l'app aspetta che Postgres sia *davvero* pronto, non solo che il processo sia partito. Risolve la race condition classica "l'app si connette prima che Postgres sia pronto e crasha".
- **Volume nominato** per i dati persistenti: gestito da Docker, sopravvive a `docker compose down` (ma muore con `down -v`).
- **`secrets` invece di env vars** per password e chiavi (vedi sezione 15). Le env sono leggibili da `docker inspect` e finiscono nei log; i secret in `/run/secrets/` no.
- **`develop.watch`** per hot-reload nativo di Compose, senza dover usare bind mount tradizionali (che su WSL2 sono problematici, vedi sezione 10).

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

**Perché preferire i profili a file separati per dev/test/prod:** unico file da mantenere allineato. Aggiungere un nuovo servizio non richiede di toccare 3 file diversi.

### 13.2 File multipli per ambienti diversi

Quando i profili non bastano (es. configurazioni *radicalmente* diverse tra dev e prod, come immagine pre-built vs build locale), pattern standard:

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

**Perché user-defined bridge è meglio del default:** sulla bridge di default Docker non avvia il DNS embedded per nome. I container possono comunicare solo via IP, che cambia ad ogni restart. Sulla user-defined, ogni container è raggiungibile per nome.

Compose crea automaticamente una user-defined bridge per ogni progetto, quindi se usi sempre Compose questo è già risolto.

### 14.2 Volumi

Tre tipi:

1. **Named volume** (gestito da Docker): `-v db_data:/var/lib/postgresql/data` → consigliato per dati persistenti.
2. **Bind mount** (cartella host): `-v $(pwd):/app` → utile in sviluppo. Su WSL2, **solo** se la cartella host è dentro il filesystem WSL (vedi sezione 10).
3. **tmpfs** (in RAM): `--tmpfs /tmp` → per dati effimeri.

**Perché named volumes per i dati di produzione:**

- Backup più semplice (Docker sa dove sono).
- Performance migliore di bind mount (specie su WSL2/macOS).
- Portabilità: non legato a un path specifico dell'host.

Comandi utili:

```bash
docker volume ls
docker volume inspect db_data
docker volume prune          # rimuove i volumi NON usati (attenzione)
```

> **Avvertenza forte su `docker compose down -v`**: il `-v` rimuove anche i volumi nominati. Se è il volume del database, **hai appena cancellato il DB**. Non è recuperabile. Usa `down` senza `-v` per fermarei container preservando i dati.

---

## 15. Gestione di variabili d'ambiente e secret

### 15.1 Regole base

- **Mai** committare il file `.env`. Committa solo `.env.example` con chiavi vuote o di esempio.
- Aggiungi `.env` e `.env.*` al `.gitignore` e al `.dockerignore`.
- In Compose, distingui:
  - `env_file:` → variabili visibili dentro il container (es. configurazione runtime)
  - file `.env` nella stessa cartella del `compose.yml` → variabili **per Compose stesso** (interpolation, es. `${APP_VERSION}`).

### 15.2 Secret in Compose con file (sviluppo locale)

Per password e chiavi, usa `secrets:` (esempio già visto in sezione 13). I secret vengono montati in `/run/secrets/<nome>` come file. **Non finiscono nelle env**, dove sarebbero leggibili da `docker inspect` e finirebbero nei log di errore di librerie chiacchierone.

Pattern per Postgres (già visto in compose):

```yaml
environment:
  POSTGRES_PASSWORD_FILE: /run/secrets/db_password   # NB: _FILE suffix
secrets:
  - db_password
```

L'immagine ufficiale `postgres` riconosce le variabili `*_FILE` e legge il valore dal path indicato. Quasi tutte le immagini ufficiali (postgres, mysql, redis, rabbitmq, mariadb) supportano questo pattern.

### 15.3 Secret a build time (BuildKit)

Per token di build (es. `npm` registry privato, GitHub PAT) **mai** scriverli con `ARG`: finiscono negli strati dell'immagine ed è banale estrarli con `docker history`. Usa BuildKit secrets:

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

**Perché:** il file viene montato solo per la durata di quel `RUN`, non viene committato in alcun layer.

### 15.4 In produzione: il ponte verso un secret manager

Tre pattern realistici, dal più semplice al più robusto.

**A) Secret iniettati dall'orchestratore (Docker Swarm / Kubernetes)**

Sia Swarm (`docker secret create`) che Kubernetes (oggetti `Secret`) gestiscono nativamente i secret e li montano nei container come file in `/run/secrets/`. Se la tua app è già strutturata per leggere da file (vedi sezione 15.2), la stessa app funziona identica in dev (file via Compose) e in prod (file via Swarm/k8s).

**B) Sidecar Vault Agent (HashiCorp Vault)**

Un container "sidecar" Vault Agent autentica al Vault, recupera i secret e li scrive in un volume `tmpfs` condiviso con l'app. L'app legge da file, non sa nulla di Vault. Esempio Compose minimo (in produzione si fa via k8s con `vault-agent-injector`):

```yaml
services:
  vault-agent:
    image: hashicorp/vault:1.16
    command: agent -config=/config/agent.hcl
    volumes:
      - ./vault-agent.hcl:/config/agent.hcl:ro
      - secrets-tmpfs:/secrets
    environment:
      VAULT_ADDR: https://vault.example.com

  app:
    image: mio-progetto/app:1.0.0
    depends_on:
      - vault-agent
    volumes:
      - secrets-tmpfs:/run/secrets:ro
    # L'app legge /run/secrets/db_password etc.

volumes:
  secrets-tmpfs:
    driver_opts:
      type: tmpfs
      device: tmpfs
```

**C) External Secrets Operator (Kubernetes)**

In K8s, l'External Secrets Operator sincronizza secret da provider esterni (AWS Secrets Manager, GCP Secret Manager, Azure Key Vault, Vault, 1Password) in `Secret` nativi K8s. L'app continua a leggere `Secret` come sempre.

**Cosa NON fare in produzione (mai):**

- Secret in chiaro in repo Git, anche se il repo è "privato".
- Secret in env vars di un Dockerfile (`ENV API_KEY=xxx`).
- Secret passati come `--build-arg` (vanno nei layer).
- Secret committati nei file `.env` di Compose.
- Secret in immagini Docker pubblicate, anche su registry "privati": un giorno il registry diventa pubblico per errore o uno degli account viene compromesso.

---

## 16. Rate limit di Docker Hub e come gestirli

Docker Hub limita le pull di immagini:

- **100 pull / 6h** per indirizzo IP anonimo
- **200 pull / 6h** per utente Docker Hub gratuito autenticato
- Limiti più alti per piani a pagamento

In CI/CD (e perfino in dev attivo se sei in 5 colleghi dietro lo stesso NAT aziendale) si raggiungono in fretta. Sintomo: `pull access denied` o `toomanyrequests: You have reached your pull rate limit`.

### 16.1 Mitigazioni in ordine di efficacia crescente

**A) Autenticati con un account Docker Hub gratuito**

```bash
docker login
```

In CI, esporta `DOCKERHUB_USERNAME` e `DOCKERHUB_TOKEN` (token, non password) come secret e fai login all'inizio del job. Raddoppia il limite a 200/6h.

**B) Usa registry alternativi quando possibile**

Le base image più popolari sono mirrorate altrove:

- `ghcr.io/...` (GitHub Container Registry) — gratuito, generoso
- `mcr.microsoft.com` (Microsoft Container Registry) per immagini Microsoft
- `public.ecr.aws/...` (Amazon ECR Public) — gratuito, generoso
- `quay.io/...` (Red Hat Quay)

Esempio: usa `mcr.microsoft.com/devcontainers/base:ubuntu` invece di `ubuntu` quando puoi.

**C) Pull-through cache (registry mirror)**

Configura il daemon perché quando chiedi `nginx:alpine`, in realtà chieda al tuo mirror locale, che lo recupera da Docker Hub solo se non lo ha. Setup classico con Harbor, Sonatype Nexus, o un semplice `registry:2` configurato come `registry-mirrors`.

```json
{
  "registry-mirrors": ["https://mirror.example.com"]
}
```

In team di una certa dimensione, questo è quasi obbligatorio.

**D) Prevenire i pull inutili**

- Pin con digest (vedi sezione 22): `docker pull` di un'immagine con digest noto e già presente è no-op.
- Cache layer in CI con BuildKit (vedi sezione 17).
- Evita `docker system prune -a` in CI: rimuove tutto e costringe a ripullare al prossimo run.

---

## 17. BuildKit avanzato e cache CI

BuildKit è il builder di default da Docker 23.0. Ti dà:

- Build parallele degli stage indipendenti
- Cache mount (`--mount=type=cache`)
- Secret a build time (`--mount=type=secret`)
- SBOM e provenance integrati
- Cache esterna remota (cruciale in CI)

### 17.1 Cache esterna in CI

In CI ogni run parte da zero: nessuna cache locale. Senza una cache esterna, ogni build riscarica e ricompila tutto. La soluzione moderna è esportare la cache su un registry o su filesystem condiviso:

```bash
docker buildx build \
  --cache-to   type=registry,ref=ghcr.io/mio-org/mio-progetto:buildcache,mode=max \
  --cache-from type=registry,ref=ghcr.io/mio-org/mio-progetto:buildcache \
  -t ghcr.io/mio-org/mio-progetto:1.0.0 \
  --push .
```

**Cosa fanno:**

- `--cache-to`: a fine build, esporta la cache (tutti gli intermedi) in un'immagine OCI nel registry.
- `--cache-from`: a inizio build, importa quella cache per saltare i layer immutati.
- `mode=max`: esporta tutti gli stage intermedi (non solo l'ultimo). Cache più grossa ma molto più efficace.

Differenza tipica: build CI da 12 minuti → 90 secondi quando cambi solo il codice applicativo e non le dipendenze.

### 17.2 Build multi-piattaforma

Per produrre immagini sia per `linux/amd64` (server) che `linux/arm64` (Apple Silicon, Raspberry Pi, AWS Graviton):

```bash
# Una tantum: crea un builder che supporta più piattaforme
docker buildx create --name multi --driver docker-container --use

# Build cross-platform via QEMU
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t mio-org/mio-progetto:1.0.0 \
  --push .
```

**Perché il driver `docker-container`:** il driver di default (`docker`) non supporta multi-platform né cache esterna avanzata. Quello `docker-container` gira come container privilegiato dedicato e abilita tutte le feature.

> **Nota:** `containerd-snapshotter` (vedi sezione 24) è richiesto per caricare immagini multi-platform nello store locale. Senza, puoi solo `--push` direttamente al registry, non `--load` in locale.

### 17.3 Bake: orchestrare build complesse

Per progetti monorepo con molte immagini, `docker buildx bake` legge un file HCL/JSON e builda tutto in parallelo con cache condivisa:

```hcl
# docker-bake.hcl
group "default" {
  targets = ["api", "web", "worker"]
}

target "_common" {
  context = "."
  cache-from = ["type=registry,ref=ghcr.io/mio-org/cache:latest"]
  cache-to   = ["type=registry,ref=ghcr.io/mio-org/cache:latest,mode=max"]
}

target "api" {
  inherits = ["_common"]
  dockerfile = "api/Dockerfile"
  tags = ["ghcr.io/mio-org/api:latest"]
}
```

```bash
docker buildx bake --push
```

---

## 18. GPU passthrough NVIDIA (opzionale, per ML/AI)

Se hai una GPU NVIDIA e vuoi usarla dai container:

### 18.1 Prerequisiti

- Driver NVIDIA aggiornati su Windows (≥ 472.39 per WSL2).
- Verifica dentro Ubuntu:

```bash
nvidia-smi
```

Se il comando funziona dentro Ubuntu, il driver passa correttamente attraverso WSL2.

### 18.2 Installa NVIDIA Container Toolkit (in Ubuntu)

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

### 18.3 Test

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

> Su Docker Desktop il GPU passthrough è abilitato automaticamente se il driver Windows è aggiornato; il toolkit non va installato manualmente nella distro WSL.

---

## 19. Dev Containers con VS Code

I **Dev Containers** ti permettono di sviluppare *dentro* il container che andrà in produzione: stessa toolchain, stesse versioni, niente "funziona sulla mia macchina".

### 19.1 Estensione

Installa in VS Code: `Dev Containers` (di Microsoft) e `WSL`.

### 19.2 `.devcontainer/devcontainer.json`

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

**Perché vale la pena:** un nuovo membro del team apre il repo, fa "Reopen in Container", e dopo 5 minuti ha esattamente la tua toolchain — stesso Node, stesso Python, stessi tool. Niente "ma a me il test passa". Niente onboarding di mezza giornata.

---

## 20. `docker context`: gestire più ambienti

`docker context` ti permette di passare velocemente tra demoni Docker diversi (locale, server remoto via SSH, CI):

```bash
docker context ls
docker context create remote --docker "host=ssh://utente@server.example.com"
docker context use remote
docker ps                  # parla con il demone remoto
docker context use default # torna al locale
```

**Perché è meglio di SSH manuale o di esporre il demone in TCP:**

- Niente `ssh` interattivo: `docker compose` e `docker buildx` funzionano transparentemente sul remoto.
- Niente esposizione del demone in TCP (insicura, vedi sezione 5.3).
- Cambio di contesto istantaneo.

Utile per deploy "manuale" su un VPS o per debug di un ambiente di staging.

---

## 21. Manutenzione: comandi quotidiani

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

> **Avvertenza forte su `docker system prune -a --volumes`**: rimuove **tutte** le immagini non in uso (anche quelle che usi raramente ma che ti costerà un'ora ripullare e ricompilare), **tutti** i container stoppati, **tutti** i volumi non riferiti, **tutta** la cache di build. È utilissimo per recuperare disco, ma è l'equivalente di un `rm -rf`. Non eseguirlo in CI se hai cache esterna che dipende da layer locali. Su workstation di sviluppo, fallo solo quando hai davvero finito spazio e sai che ti tocca rebuildare tutto.

### 21.1 Recupero spazio del VHDX (se non hai abilitato `sparseVhd`)

Su WSL2 il file VHDX cresce e (senza `sparseVhd=true`) non restituisce spazio a Windows. Per farlo manualmente, da PowerShell amministratore, dopo aver fatto `docker system prune`:

```powershell
wsl --shutdown

# Il path dipende da come hai installato Docker:
# - Docker Desktop recente: lo storage è dentro la distro WSL
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

Se hai abilitato `sparseVhd=true` in `.wslconfig` (sezione 3.1), questo passaggio è **automatico** dopo qualche tempo (non immediato — può richiedere ore di idle).

---

## 22. Sicurezza: hardening, SBOM, provenance, scansione, firma

Un professionista applica almeno questi controlli.

### 22.1 Regole di base

1. **Non eseguire mai container come root** se non strettamente necessario (`USER app` nel Dockerfile).
2. **Usa immagini ufficiali** o verificate (Docker Official, Verified Publisher).
3. **Pin delle versioni**: mai `:latest` in produzione. Usa il tag major+minor (`postgres:16.4`) e, idealmente, il **digest**.
4. **Filesystem read-only** quando l'app non deve scrivere (`read_only: true`).
5. **Drop di tutte le capability** e re-aggiunta solo di quelle necessarie (`cap_drop: ALL`, `cap_add: NET_BIND_SERVICE`).
6. **Niente `--privileged`** se non strettamente indispensabile.
7. **Mai esporre il socket** Docker (`/var/run/docker.sock`) dentro un container, salvo casi consapevoli (CI/CD): equivale a dare root all'host.
8. **Limita le porte** con bind esplicito a `127.0.0.1` se il servizio non deve essere raggiungibile dalla rete.
9. **Aggiorna regolarmente**: `docker pull` periodico delle base image, ricostruzione settimanale delle immagini.

### 22.2 Pin con digest (riproducibilità reale)

```dockerfile
FROM node:20.14-alpine@sha256:7c3b2... AS build
```

Il digest è immutabile: anche se domani il tag `20.14-alpine` viene ripubblicato con CVE diverse, la tua build pesca esattamente lo stesso layer. Strumenti come **Renovate** o **Dependabot** automatizzano l'aggiornamento dei digest pin (e ti aprono PR quando ci sono fix di sicurezza disponibili).

**Perché il digest e non solo il tag:** un tag è una label mutabile, un digest è un hash crittografico immutabile. `node:20.14-alpine` oggi e tra 6 mesi possono essere immagini diverse. `node:20.14-alpine@sha256:...` no.

### 22.3 Scansione CVE

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

> **Nota su `docker scan`**: il vecchio `docker scan` (basato su Snyk) è **deprecato e rimosso** dalle versioni recenti di Docker. Usa `docker scout` (sostituto ufficiale) o uno scanner esterno come Trivy/Grype.

### 22.4 SBOM (Software Bill of Materials) e provenance

Generare l'SBOM diventa requisito di compliance (NIST SSDF, EU Cyber Resilience Act). BuildKit lo fa nativamente:

```bash
docker buildx build \
  --sbom=true \
  --provenance=mode=max \
  -t mio-progetto/app:1.0.0 \
  --push .
```

**Cosa contengono:**

- **SBOM**: elenco di tutti i pacchetti software dentro l'immagine (versioni, hash, licenze). Quando esce una nuova CVE, sai immediatamente se sei vulnerabile senza dover riscandire.
- **Provenance** (mode=max): cosa, chi, dove, quando, con quale Dockerfile, con quali sorgenti è stata costruita questa immagine. Audit trail completo.

Per generare un SBOM standalone con Syft:

```bash
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  anchore/syft:latest mio-progetto/app:1.0.0 -o spdx-json > sbom.spdx.json
```

### 22.5 Firma delle immagini (Cosign)

Se le pubblichi su un registry, firmale:

```bash
cosign generate-key-pair
cosign sign --key cosign.key registry.example.com/mio-progetto/app:1.0.0
cosign verify --key cosign.pub registry.example.com/mio-progetto/app:1.0.0
```

Cosign supporta anche firma **keyless via OIDC** (GitHub Actions, GitLab CI), che è il pattern moderno: niente chiavi private da gestire, l'identità è quella della pipeline CI:

```bash
# In GitHub Actions, con OIDC abilitato:
cosign sign ghcr.io/mio-org/mio-progetto@sha256:...
```

> **Nota su Docker Content Trust / Notary v1**: il vecchio meccanismo `DOCKER_CONTENT_TRUST=1` con Notary v1 è **deprecato**. È in fase di smantellamento. Usa Cosign / Sigstore / Notary v2: sono lo standard moderno.

### 22.6 Linting di Compose

```bash
docker compose config            # valida il file e mostra il merge finale
docker compose config --services # elenca solo i servizi
```

In CI, far fallire il job se `docker compose config` non passa è un guard rail minimo gratuito.

---

## 23. Rootless mode (avanzato, opzionale)

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

**Perché rootless:** il demone Docker tradizionale gira come root e chiunque sia nel gruppo `docker` ha de facto root sull'host. In rootless mode il demone gira come utente normale, dentro un namespace utente Linux. Anche se lo compromettono, non hanno root.

**Limitazioni che devi accettare:**

- Porte < 1024 richiedono `setcap` o reverse proxy.
- Alcune feature di rete (es. `--net=host` cross-namespace) non sono disponibili.
- Performance leggermente peggiori della modalità rooted (overhead del namespace utente).
- Alcuni storage driver non supportati (devi usare `fuse-overlayfs` invece di `overlay2` "rooted").

In dev su laptop personale è perfetto. Su server di produzione molto carichi, valuta caso per caso.

---

## 24. Tuning avanzato del daemon (opzionale)

Le opzioni qui sotto sono potenti ma richiedono consapevolezza. **Non** aggiungerle al `daemon.json` solo perché "è più professionale": ognuna ha un trade-off.

### 24.1 `userland-proxy: false`

```json
{
  "userland-proxy": false
}
```

**Cosa fa:** disabilita il piccolo proxy in user space che Docker usa per inoltrare le porte pubblicate. Il traffico va direttamente via iptables/nftables. Migliore performance e meno consumo di memoria (un processo `docker-proxy` in meno per ogni porta pubblicata).

**Perché può rompersi:** su WSL2 con Docker Desktop ha causato in passato regressioni di raggiungibilità delle porte da Windows verso il container. Su Linux nativo è più affidabile. Se su WSL2 le porte non funzionano dopo l'attivazione, riportala a `true` e chiudi il caso.

### 24.2 `containerd-snapshotter: true`

```json
{
  "features": {
    "containerd-snapshotter": true
  }
}
```

**Cosa fa:** sostituisce il vecchio storage driver Docker con quello di containerd. Sblocca:

- Caricamento di immagini multi-piattaforma nello store locale (es. `docker buildx --load --platform linux/amd64,linux/arm64`).
- SBOM e provenance attaccati alle immagini visibili in locale.
- Compatibilità più diretta con altri tool dell'ecosistema OCI (nerdctl, BuildKit standalone).

> **Avvertenza grossa:** attivarlo su un'installazione **esistente** invalida lo store delle immagini esistente. **Non perdi dati né volumi**, ma le immagini scaricate vanno ripullate. Pianifica l'attivazione per un momento "comodo" (e magari fai un `docker images > my-images.txt` prima, per ricordarti cosa avevi).

### 24.3 `icc: false`

```json
{
  "icc": false
}
```

**Cosa fa:** disabilita la inter-container communication sulla bridge di default. Container sulla stessa bridge non si vedono finché non li metti su una rete user-defined o non aggiungi `--link`.

**Perché può confondere:** se tipicamente lavori con `docker run` di servizi multipli senza creare una rete dedicata, smettono di parlarsi e ti sembra rotto. Compose è immune perché crea sempre una rete user-defined per progetto.

Aggiungilo solo se hai imparato l'igiene di "una rete user-defined per ogni gruppo di container che devono parlarsi".

### 24.4 `default-runtime` e runtime alternativi

```json
{
  "default-runtime": "runc",
  "runtimes": {
    "youki": { "path": "/usr/local/bin/youki" }
  }
}
```

Per ambienti con esigenze particolari (es. `kata-containers` per isolation hardware-level, `gvisor` per sandboxing, `youki` runtime in Rust). Argomento avanzato, fuori scope di una guida introduttiva.

---

## 25. Aggiornamenti

### 25.1 Docker Desktop

- L'app notifica gli aggiornamenti. Aggiorna **almeno una volta al mese** per ricevere patch di sicurezza.
- Prima di un major update, fai `docker system df` e considera un backup dei volumi importanti (sezione 26).

### 25.2 Docker Engine in Ubuntu

```bash
sudo apt-get update
sudo apt-get upgrade -y \
  docker-ce docker-ce-cli containerd.io \
  docker-buildx-plugin docker-compose-plugin
```

### 25.3 Aggiornamento delle base image

Per i progetti che ti interessano davvero, **non** fare aggiornamento massivo a tappeto: fai un elenco esplicito delle base image che mantieni e ripullale.

```bash
# Esempio mirato (consigliato)
for img in \
  node:20.14-alpine \
  postgres:16.4-alpine \
  nginx:1.27-alpine; do
  docker pull "$img"
done

# Ricostruisci le immagini dei tuoi progetti
docker compose build --pull --no-cache
```

**Perché evitare il pull massivo di "tutte le immagini locali":** ti ritrovi a ripullare immagini sperimentali, vecchi tag che non usi più, immagini da progetti dismessi. Spreca banda, riempie disco e contribuisce ai rate limit di Docker Hub.

In CI puoi automatizzare il pull mirato con un cron settimanale e PR automatica per gli aggiornamenti di digest tramite Renovate/Dependabot.

---

## 26. Backup dei dati persistenti

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

### 26.1 Backup logico per database

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

**Perché preferire il dump logico:** un backup del volume a freddo (con il DB acceso) può catturare uno stato inconsistente — tipo, una transazione a metà — e un restore può corrompersi. `pg_dump` parla con Postgres e ottiene uno snapshot consistente *while it's running*.

### 26.2 Backup del `.wslconfig` e dell'intera distro

Per non rifare tutto se cambi PC:

```powershell
# Esporta l'intera distro Ubuntu
wsl --export Ubuntu D:\backup\ubuntu-2026-05-07.tar

# Su un'altra macchina:
wsl --import Ubuntu C:\WSL\Ubuntu D:\backup\ubuntu-2026-05-07.tar --version 2
```

E copia anche `C:\Users\<utente>\.wslconfig` e `~/.docker/config.json`.

---

## 27. Logging centralizzato

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

### 27.1 Driver bloccanti vs non-bloccanti

**Importante:** alcuni driver di log sono **bloccanti** di default. Se il backend dei log è giù o lento, il container si blocca sul `stdout` (l'app rallenta o si pianta). Per ambienti di produzione, valuta `mode: non-blocking` con buffer adeguato:

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

**Trade-off:** in `non-blocking`, sotto pressione i log più vecchi del buffer vengono **persi** (non bloccano l'app, ma sono persi). Per molti workload è il compromesso giusto. Per workload audit-critical, devi accettare il blocking e dimensionare il backend di logging adeguatamente.

---

## 28. Troubleshooting rapido

| Sintomo | Causa probabile | Soluzione |
|---|---|---|
| `docker: command not found` in Ubuntu | Integrazione WSL non attiva o PATH non rinfrescato | Settings → Resources → WSL Integration; riapri il terminale |
| `permission denied` su `/var/run/docker.sock` | Utente non nel gruppo `docker` | `sudo usermod -aG docker $USER` + `wsl --shutdown` |
| `Cannot connect to the Docker daemon` | Demone non avviato | `sudo systemctl start docker` o avvia Docker Desktop |
| Container lentissimi su bind mount Windows | Stai montando da `/mnt/c/...` | Sposta il progetto dentro il filesystem WSL (`~/progetti/...`) |
| Disco Windows si riempie | VHDX di WSL/Docker cresciuto | Abilita `sparseVhd=true`, oppure compatta a mano (sezione 21.1) |
| `pull access denied` o `toomanyrequests` | Rate limit Docker Hub | `docker login` o registry mirror (sezione 16) |
| Build sempre da capo | Cache invalidata da `COPY .` prima di `npm ci` | Copia prima `package*.json`, poi installa, poi copia il resto |
| `bash\r: No such file or directory` | Script con line ending CRLF | Vedi sezione 3.3 (`.gitattributes` + `git add --renormalize .`) |
| `x509: certificate has expired or is not yet valid` su `docker pull` | Orologio di WSL2 sfasato | `sudo hwclock -s` o installa `systemd-timesyncd` (sezione 3.4) |
| Le porte dei container non si vedono da Windows | Modalità di rete WSL2 non ottimale | Prova `networkingMode=mirrored` in `.wslconfig` (sezione 3.1) |
| Le porte funzionavano e dopo aggiornamento no | `userland-proxy: false` su Docker Desktop con WSL2 | Riportalo a `true` (sezione 24.1) |
| DNS che non risolve dentro i container in azienda | VPN/proxy aziendale | `--dns=8.8.8.8` su `docker run` o `dns:` in compose; oppure `dns` in `daemon.json` |
| `docker compose up` fallisce con "no such service" su un profilo | Manca `--profile <nome>` | `docker compose --profile dev up` |
| `failed to register layer ... no space left on device` | Volume di Docker pieno | `docker system prune -af --volumes` poi `compact vdisk` |
| `iptables: No chain/target/match by that name` | Firewall in WSL2 obsoleto | `sudo update-alternatives --set iptables /usr/sbin/iptables-legacy` |
| Container che reagisce male a Ctrl+C | Manca init/PID 1 sano | Aggiungi `tini` nel Dockerfile o `init: true` nel compose |
| Immagini scomparse dopo abilitazione `containerd-snapshotter` | Cambio storage driver | Vedi sezione 24.2: vanno ripullate, è normale |
| `denied: requested access to the resource is denied` su `docker push` | Non sei loggato al registry o non hai permessi | `docker login <registry>` |

---

## 29. Prossimi passi consigliati

Una volta padroneggiate le basi, approfondisci in quest'ordine:

1. **BuildKit avanzato in CI**: cache esterna su registry (sezione 17), build multi-piattaforma con `buildx`, `bake` per monorepo.
2. **Registry privato** (Harbor, GitHub Container Registry, GitLab Container Registry) con CI/CD (GitHub Actions, GitLab CI) che builda, firma con Cosign, scansiona con Trivy e pubblica.
3. **Compose Profiles e `develop.watch`** in profondità, e migrazione dei "vecchi" `docker-compose.dev.yml` ai profili.
4. **Orchestrazione**: Docker Swarm per iniziare (Compose-compatible), poi Kubernetes (k3d/kind in locale) quando ti servirà davvero.
5. **Osservabilità**: Prometheus + Grafana + Loki + cAdvisor per metriche e log dei container.
6. **Policy as code**: scansione automatica con Trivy/Scout in pipeline, blocco di immagini con CVE critiche, controllo SBOM con Open Policy Agent / Conftest.
7. **Supply chain security**: Cosign keyless via OIDC, attestation in-toto, Sigstore.
8. **Secret in produzione**: Vault Agent o External Secrets Operator (vedi sezione 15.4).

---

## 30. Checklist del professionista

Considera l'installazione completata solo quando puoi spuntare **tutte** queste voci:

### Sistema

- [ ] Virtualizzazione attiva nel BIOS/UEFI
- [ ] WSL2 aggiornato (`wsl --update`), Ubuntu in versione 2, systemd attivo
- [ ] `.wslconfig` configurato con limiti RAM/CPU sensati e (se supportato) `networkingMode=mirrored`, `sparseVhd=true`, `autoMemoryReclaim=gradual` nelle sezioni corrette per la tua versione di WSL
- [ ] `git config core.autocrlf input` e `.gitattributes` con `eol=lf` di default

### Docker

- [ ] Docker Desktop **oppure** Docker Engine installato e funzionante
- [ ] Integrazione WSL2 attiva sulla distro Ubuntu (se usi Desktop)
- [ ] `docker run --rm hello-world` funziona dentro Ubuntu **senza `sudo`**
- [ ] `daemon.json` configurato con log rotation, address pools, live-restore, no-new-privileges, BuildKit
- [ ] BuildKit, Compose v2 e Buildx disponibili (`docker buildx version`, `docker compose version`)
- [ ] Hai deciso consapevolmente se attivare `containerd-snapshotter` e `userland-proxy: false` (sezione 24)

### Workflow

- [ ] Capisci la differenza tra immagine, container, volume e rete
- [ ] I tuoi progetti vivono dentro `~/progetti/` (filesystem WSL), **non** in `/mnt/c/...`
- [ ] Sai scrivere un `Dockerfile` multi-stage con utente non-root, init (`tini` o `init: true` Compose), healthcheck, cache mount
- [ ] Lint di Dockerfile con `hadolint` integrato nel tuo workflow
- [ ] Sai scrivere un `docker-compose.yml` con limiti di risorse, healthcheck, reti dedicate, volumi nominati, secret, profiles e `develop.watch`
- [ ] Hai una `.devcontainer/devcontainer.json` per VS Code

### Sicurezza e operations

- [ ] Pin delle base image a major+minor (`node:20.14-alpine`), digest in produzione
- [ ] Strategia anti-rate-limit Docker Hub definita (login + opzionalmente mirror)
- [ ] Cache esterna BuildKit configurata in CI (`--cache-to`/`--cache-from` su registry)
- [ ] Scansione CVE in CI (Scout o Trivy) che blocca il merge su HIGH/CRITICAL con fix
- [ ] SBOM e provenance generati in build (`buildx --sbom --provenance=mode=max`)
- [ ] Workflow di pulizia periodica (`docker system df` / `prune` mirato)
- [ ] Piano di backup per i volumi importanti (dump logico per i DB)
- [ ] Strategia di logging definita (json-file con rotazione in dev, driver centralizzato in prod, considerato `mode: non-blocking`)
- [ ] Strategia secret prod definita (Swarm/k8s native, Vault Agent o External Secrets)
- [ ] Conosci `docker context` per gestire ambienti remoti senza esporre il demone

Quando tutte le caselle sono spuntate, puoi considerarti operativo a livello professionale.
