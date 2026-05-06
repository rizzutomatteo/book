# Configurazione Professionale di GitHub — Guida Completa

> Guida completa per configurare GitHub e Git da zero in modo professionale, sicuro ed efficiente. Pensata per chi programma e vuole lavorare con gli stessi standard di un team senior. Ogni passo è accompagnato dalla spiegazione del *perché*.

---

## Premessa importante: Windows nativo vs WSL

Se sei uno **sviluppatore**, la scelta professionale è **WSL2 con Ubuntu**, non Windows nativo. Quasi tutto l'ecosistema dev moderno (Node, Python, Docker, Rust, Go, toolchain Linux varie) gira nativamente su Linux: usarlo via WSL elimina mille piccoli attriti (path, line ending, permessi, performance di npm/pip su NTFS, container che girano nativamente).

Linea guida che seguirò in questa guida:

- **Tutto il lavoro di sviluppo si fa in WSL.** Git, SSH, chiavi, signing, repo: tutto dentro Ubuntu.
- **Windows nativo si configura comunque**, perché ti capiterà di clonare repo non-dev (config, dotfile, progetti scolastici, cose veloci) o di lavorare con strumenti Windows-only.
- Dove i comandi differiscono tra i due ambienti, troverai entrambe le versioni etichettate **Windows (PowerShell)** e **WSL (Ubuntu/bash)**.

Se non sei uno sviluppatore (es. usi GitHub solo per appunti, documenti, progetti personali leggeri), puoi ignorare la parte WSL e seguire solo la colonna Windows.

---

## Indice

1. [Creazione dell'account GitHub](#1-creazione-dellaccount-github)
2. [Sicurezza dell'account (priorità massima)](#2-sicurezza-dellaccount-priorità-massima)
3. [Profilo pubblico professionale](#3-profilo-pubblico-professionale)
4. [Configurazione di GitHub via browser](#4-configurazione-di-github-via-browser)
5. [Installazione del software (Windows + WSL)](#5-installazione-del-software-windows--wsl)
6. [Identità Git locale](#6-identità-git-locale)
7. [Chiave SSH per autenticazione](#7-chiave-ssh-per-autenticazione)
8. [Commit verificati con SSH signing](#8-commit-verificati-con-ssh-signing)
9. [Configurazioni Git "quality of life"](#9-configurazioni-git-quality-of-life)
10. [Diff e merge tool con VS Code](#10-diff-e-merge-tool-con-vs-code)
11. [Alias Git utili](#11-alias-git-utili)
12. [Gitignore globale](#12-gitignore-globale)
13. [Credential helper per HTTPS](#13-credential-helper-per-https)
14. [Configurazione della shell](#14-configurazione-della-shell)
15. [GitHub CLI](#15-github-cli)
16. [Integrazione con VS Code](#16-integrazione-con-vs-code)
17. [Verifica finale](#17-verifica-finale)
18. [Best practice per i tuoi repository](#18-best-practice-per-i-tuoi-repository)
19. [Cheat sheet completo](#19-cheat-sheet-completo)

---

## 1. Creazione dell'account GitHub

Vai su [github.com](https://github.com) e clicca **Sign up**.

**Email da usare.** Usa un indirizzo email *personale* a cui avrai accesso a vita, non un'email scolastica o aziendale. *Perché:* l'email scolastica viene disattivata dopo la laurea e quella aziendale alla fine del rapporto di lavoro; perdere l'accesso all'email significa rischiare di perdere l'account e tutti i tuoi contributi pubblici. Potrai sempre collegare un account Google in un secondo momento per il login rapido.

**Scelta dello username.** È il tuo biglietto da visita per recruiter, colleghi e contributori open source: appare in ogni URL dei tuoi repo (`github.com/tuousername/...`) e in ogni commit. Regole:

- Solo nome e cognome (es. `mariorossi`, `mario-rossi`, `mrossi`).
- Niente numeri casuali, nickname da gaming, date di nascita.
- Ammessi: lettere, numeri e trattini singoli. Vietati spazi e caratteri speciali.
- Massimo 39 caratteri.

*Perché:* uno username serio comunica professionalità. `xXmario99Xx` non viene mai assunto.

**Verifica l'email** cliccando il link di conferma. Senza verifica email non puoi né pushare né essere riconosciuto come autore dei commit.

---

## 2. Sicurezza dell'account (priorità massima)

GitHub contiene il tuo lavoro, le tue credenziali (token, API key nei `.env` dimenticati) e il tuo nome. Un account compromesso è un disastro. Fai questi passi *prima* di iniziare a lavorare.

### 2.1 Attiva l'autenticazione a due fattori (2FA)

Vai su **Settings → Password and authentication → Two-factor authentication**.

Scegli **Authenticator app**, **non SMS**. *Perché:* gli SMS sono vulnerabili al SIM swapping; un'app TOTP come Google Authenticator, Microsoft Authenticator, Authy, 1Password o Bitwarden è offline e molto più sicura.

Scansiona il QR code con la tua app e inserisci il codice a 6 cifre per confermare.

### 2.2 Salva i recovery codes (passo più importante della guida)

Dopo l'attivazione del 2FA, GitHub ti mostra **16 recovery codes**. Salvali subito.

**Dove salvarli:**

- In un password manager (Bitwarden, 1Password, KeePass).
- Stampati e conservati in un posto fisico sicuro.
- **Mai** in chiaro su Desktop, in note di Telegram a te stesso, in email.

*Perché:* se perdi/rompi/resetti il telefono con l'app authenticator e non hai i recovery codes, **perdi l'account in modo definitivo**. GitHub non ha un servizio clienti che ti riapre l'account: la chiave sei tu. Questa è la causa numero uno di account persi.

### 2.3 Aggiungi una passkey

In **Settings → Password and authentication → Passkeys** aggiungi una passkey (Windows Hello, Touch ID, YubiKey). Permette login senza password e resistente al phishing — è la frontiera della sicurezza moderna.

### 2.4 Email di backup

In **Settings → Emails** aggiungi una seconda email come backup. Se perdi accesso alla principale, hai una via di recupero in più.

### 2.5 Privacy delle email nei commit

In **Settings → Emails** attiva:

- **Keep my email addresses private**
- **Block command line pushes that expose my email**

GitHub ti darà un'email "noreply" del tipo `12345+username@users.noreply.github.com` da usare nei commit. *Perché:* ogni commit pubblico contiene l'email dell'autore in chiaro per sempre nella history; usando l'email noreply eviti spam, harvesting da bot e fughe accidentali della tua email personale. **Annota da qualche parte questa email**, ti servirà più avanti.

---

## 3. Profilo pubblico professionale

Vai su **Settings → Public profile**.

- **Nome:** il tuo nome reale (es. "Mario Rossi"). Recruiter cercano persone, non handle.
- **Foto:** una foto professionale, ben illuminata, viso visibile. Stesso standard di LinkedIn.
- **Bio:** una riga con cosa fai e che lavoro ti interessa.
- **Pronouns, location, sito, social:** opzionali. Se li metti, tienili aggiornati.

### Profile README (carta da visita pubblica)

Crea un repository con **lo stesso identico nome del tuo username** (es. se ti chiami `mariorossi`, il repo si chiama `mariorossi`). Aggiungi un file `README.md` al suo interno: GitHub lo mostrerà automaticamente in cima alla tua pagina profilo.

*Perché:* è la prima cosa che vede chi visita il tuo profilo. Mettici una breve presentazione, le tecnologie su cui lavori e qualche progetto in evidenza. Niente badge stile "I'm a passionate developer 🚀✨" — banale. Punta a contenuto vero: cosa fai, su cosa stai lavorando, come contattarti.

### Pinned repositories

Sotto la tua griglia di repo nel profilo, clicca **Customize your pins** e seleziona fino a 6 repo da mostrare in evidenza. Scegli i tuoi progetti migliori, non i più recenti per forza.

---

## 4. Configurazione di GitHub via browser

Buona parte delle impostazioni che ti differenziano da uno utente casuale sono nel pannello web. Vai su **Settings** e configura quanto segue.

### 4.1 Aspetto

**Settings → Appearance.**

- **Theme:** "Sync with system" oppure dark/light fisso a seconda di cosa preferisci.
- **Tab size preference:** se hai una preferenza per la visualizzazione del codice (2 o 4 spazi), impostala.

### 4.2 Notifiche

**Settings → Notifications.** Questa è la sezione che la maggior parte delle persone configura male e si ritrova con la inbox piena di rumore.

Configurazione consigliata:

- **Default notifications email:** la tua email principale.
- **Automatically watch repositories:** **disattiva**. *Perché:* di default GitHub ti iscrive a tutti i repo a cui contribuisci o che crei: dopo poco tempo ricevi 200 notifiche al giorno di robe che non ti interessano.
- **Automatically watch teams:** disattiva per lo stesso motivo.
- **Participating, @mentions and custom:** lascia attive email e web.
- **Watching:** lascia solo Web (eviti email di rumore).
- **Dependabot alerts:** **email + web**. Vuoi sapere subito quando c'è una vulnerabilità nelle dipendenze.

In sintesi: ricevi email solo per cose che ti riguardano direttamente (mention, partecipazione attiva, alert di sicurezza), il resto va sul tab Notifications del web.

### 4.3 Privacy e visibilità del profilo

**Settings → Public profile.** In fondo:

- **Private contributions:** scegli se mostrare i contributi a repo privati nel grafico verde. Per progetti aziendali confidenziali, attivalo a "Show private contributions".

**Settings → Account → Profile.** Considera se attivare **Achievements** e **Highlights** — sono i badge che appaiono sul profilo. Niente di cruciale.

### 4.4 Default branch globale per i nuovi repo

**Settings → Repositories → Repository default branch.**

Imposta **`main`** come default. *Perché:* è lo standard moderno; nuovi repo creati via web partiranno con `main`.

### 4.5 Sessioni e log di sicurezza

**Settings → Sessions.** Vedi tutti i dispositivi attualmente loggati. Se ne vedi uno che non riconosci, fai logout subito e cambia password.

**Settings → Security log.** Cronologia di tutto quello che è successo sull'account: login, modifiche, autorizzazioni, push. Controllala periodicamente. Se vedi qualcosa di sospetto, agisci subito.

### 4.6 Applicazioni autorizzate

**Settings → Applications → Authorized OAuth Apps** e **Authorized GitHub Apps.**

Periodicamente fai pulizia: revoca l'accesso a tool che non usi più. Ogni app autorizzata ha permessi sui tuoi repo: meno ne hai attive, minore è la superficie d'attacco.

### 4.7 Personal Access Token (quando ti servono)

**Settings → Developer settings → Personal access tokens → Fine-grained tokens.**

I PAT servono quando uno script, una CI o uno strumento ha bisogno di accedere a GitHub via HTTPS senza essere te interattivamente. Regole:

- **Sempre fine-grained**, mai "classic". I fine-grained ti permettono di limitare il token a repo specifici e a permessi specifici.
- **Scadenza breve** (30-90 giorni). Mai "no expiration".
- **Permessi minimi necessari.** Se serve solo per leggere issue, dai solo `Issues: Read`.
- **Salvali nel password manager** appena creati: GitHub li mostra una volta sola.

*Perché:* un PAT compromesso = accesso ai tuoi repo. Limitarne portata e durata limita il danno.

### 4.8 Saved replies

**Settings → Saved replies.**

Crea risposte preconfezionate per messaggi ricorrenti su issue e PR (es. "Grazie per il contributo, però manca il test", "Dovresti aprire un'issue prima"). *Perché:* manutenere un progetto open source significa rispondere alle stesse cose 100 volte; con saved replies risparmi ore.

### 4.9 SSH e GPG keys

**Settings → SSH and GPG keys.** Le configurerai dal terminale nei prossimi passi. Tienila aperta in una tab.

### 4.10 Codespaces (se userai)

**Settings → Codespaces.**

- **Editor preference:** VS Code (Web o Desktop).
- **Default region:** quella geograficamente più vicina (per latenza).
- **Dotfiles:** se hai un repo `dotfiles` con la tua configurazione (alias bash, settings VS Code, ecc.), abilitalo qui — ogni nuovo Codespace partirà già configurato a modo tuo.

### 4.11 Following

**Search →** segui sviluppatori, organizzazioni e progetti che ti interessano. Il tuo feed personale (homepage di GitHub) diventa un filtro di qualità di cose pertinenti, non rumore.

---

## 5. Installazione del software (Windows + WSL)

### 5.1 Installazione di WSL2 con Ubuntu

Apri **PowerShell come amministratore** e installa WSL:

```powershell
wsl --install -d Ubuntu
```

Riavvia il PC quando richiesto. Al primo avvio di Ubuntu, ti chiede uno username Linux e una password. Tienili semplici (questo username è solo per il tuo PC, non c'entra con quello di GitHub) ma sicuri.

Aggiorna subito il sistema:

```bash
sudo apt update && sudo apt upgrade -y
```

*Perché WSL2 invece di Windows nativo:* line ending coerenti (LF), permessi POSIX, performance Docker/Node/Python di un altro pianeta, toolchain Linux native, ambiente identico a quello di produzione (i server girano Linux). Praticamente tutti gli sviluppatori senior su Windows lavorano in WSL.

**Regola critica:** tieni il codice in `/home/tuoutente/` (filesystem Linux), **non** in `/mnt/c/Users/...` (filesystem Windows). Lavorare su `/mnt/c` da WSL è 10-100 volte più lento per operazioni Git e `npm install`.

### 5.2 Installazione di Git, VS Code, GitHub CLI

**Su Windows (PowerShell come admin):**

```powershell
winget install --id Git.Git -e
winget install --id Microsoft.VisualStudioCode -e
winget install --id GitHub.cli -e
```

**Su WSL (Ubuntu/bash):**

```bash
# Git
sudo apt install -y git

# GitHub CLI (repo ufficiale)
(type -p wget >/dev/null || (sudo apt update && sudo apt-get install wget -y)) \
  && sudo mkdir -p -m 755 /etc/apt/keyrings \
  && wget -qO- https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null \
  && sudo chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg \
  && echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
  && sudo apt update \
  && sudo apt install gh -y
```

VS Code lo installi solo su Windows: lo userai per aprire i progetti dentro WSL via l'estensione **Remote - WSL** (vedi sezione 16). *Perché:* avere due VS Code (uno Windows e uno Linux) crea solo confusione, e VS Code Windows con Remote-WSL è esattamente equivalente a uno installato in Linux.

Verifica:

```powershell
# Windows
git --version
code --version
gh --version
```

```bash
# WSL
git --version
gh --version
```

---

## 6. Identità Git locale

Git deve sapere chi sei: ogni commit viene firmato con nome ed email. Va impostato **separatamente in ogni ambiente** (Windows e WSL hanno configurazioni Git indipendenti).

**Su Windows (PowerShell):**

```powershell
git config --global user.name "Mario Rossi"
git config --global user.email "12345+mariorossi@users.noreply.github.com"
```

**Su WSL (bash):**

```bash
git config --global user.name "Mario Rossi"
git config --global user.email "12345+mariorossi@users.noreply.github.com"
```

*Perché:* nome ed email appaiono in ogni commit, in `git log`, su GitHub e nelle storie dei progetti open source a cui contribuirai. L'email **deve corrispondere** a una verificata sul tuo account GitHub, altrimenti i commit non vengono associati al tuo profilo. Usa l'email noreply del passo 2.5 per non esporre la tua email reale.

---

## 7. Chiave SSH per autenticazione

Le credenziali SSH sostituiscono la password per push, pull e clone. Sono più sicure e non si digitano ad ogni operazione.

**Best practice professionale: una chiave per ambiente.** Genera una chiave dentro Windows e una dentro WSL. *Perché:* sono ambienti distinti con file system distinti; tenere chiavi separate ti permette di revocarne una sola se uno dei due ambienti viene compromesso, senza dover risetuppare anche l'altro. Le chiavi non vanno condivise tra ambienti.

### 7.1 Genera la chiave

**Su Windows (PowerShell):**

```powershell
ssh-keygen -t ed25519 -C "12345+mariorossi@users.noreply.github.com"
```

Quando ti chiede dove salvare, premi Invio (default: `~/.ssh/id_ed25519`, che su Windows è `C:\Users\TuoUtente\.ssh\id_ed25519`).

**Su WSL (bash):**

```bash
ssh-keygen -t ed25519 -C "12345+mariorossi@users.noreply.github.com"
```

Default WSL: `/home/tuoutente/.ssh/id_ed25519`.

In entrambi i casi:

- **`-t ed25519`:** algoritmo moderno, più veloce e più sicuro di RSA. Da preferire sempre.
- **`-C "..."`:** etichetta che apparirà nel commento della chiave (l'email è la convenzione).

**Quando chiede la passphrase, INSERISCINE UNA.** Mai vuota.

*Perché:* la chiave privata è un file sul tuo disco. Se ti rubano il PC o un malware ne fa l'esfiltrazione, senza passphrase chi ha il file ha accesso completo a tutti i tuoi repo (inclusa la possibilità di pushare codice malevolo a tuo nome). Con la passphrase serve anche quella per usarla. Salvala nel password manager.

### 7.2 Configura ssh-agent

`ssh-agent` è un processo che tiene la tua chiave SSH decifrata in memoria: invece di digitare la passphrase ad ogni operazione Git, la inserisci una volta sola e l'agente la ricorda per te.

**Su Windows:**

PowerShell **come amministratore**:
```powershell
Get-Service -Name ssh-agent | Set-Service -StartupType Automatic
Start-Service ssh-agent
```
PowerShell normale:
```powershell
ssh-add $env:USERPROFILE\.ssh\id_ed25519
```

**Su WSL:**

Installa `keychain`, che mantiene la chiave caricata tra una shell e l'altra finché non riavvii WSL:
```bash
sudo apt install -y keychain
```
Aggiungi al tuo `~/.bashrc` (o `~/.zshrc`):
```bash
eval $(keychain --eval --quiet ~/.ssh/id_ed25519)
```
Ricarica:
```bash
source ~/.bashrc
```

### 7.3 Aggiungi la chiave pubblica a GitHub

**Su Windows (PowerShell):**

```powershell
Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub | Set-Clipboard
```

**Su WSL (bash):**

```bash
# Se hai installato wl-clipboard o xclip
cat ~/.ssh/id_ed25519.pub | clip.exe
# Alternativa: stampala e copiala manualmente
cat ~/.ssh/id_ed25519.pub
```

*Nota:* `clip.exe` è il comando di Windows accessibile da WSL — funziona per copiare negli appunti di Windows.

Su GitHub: **Settings → SSH and GPG keys → New SSH key**.

- **Title:** un nome riconoscibile per ogni chiave. Convenzione consigliata: `Laptop personale - Windows` e `Laptop personale - WSL Ubuntu`.
- **Key type:** `Authentication Key`.
- **Key:** incolla.

Ripeti per la seconda chiave. A questo punto su GitHub vedrai due chiavi di tipo Authentication, una per ambiente.

### 7.4 Verifica la connessione

Da entrambi gli ambienti:

```bash
ssh -T git@github.com
```

Risposta attesa: `Hi mariorossi! You've successfully authenticated, but GitHub does not provide shell access.` Il "but GitHub does not provide shell access" è normale e significa che ha funzionato.

---

## 8. Commit verificati con SSH signing

I "Verified commits" mostrano un badge verde su GitHub e garantiscono che il commit è effettivamente stato firmato con la tua chiave. Senza firma, **chiunque può configurare Git con il tuo nome ed email e committare a tuo nome**: l'autore di un commit Git non è autenticato di default. Questa è una vulnerabilità reale.

Useremo la **stessa chiave SSH** del passo precedente per firmare anche i commit. Non serve creare GPG: SSH signing è supportato da Git 2.34+ ed è la pratica moderna consigliata da GitHub stesso.

### 8.1 Aggiungi le chiavi anche come Signing Key su GitHub

Su GitHub: **Settings → SSH and GPG keys → New SSH key**. Per **ognuna** delle due chiavi pubbliche (Windows e WSL), aggiungila una seconda volta selezionando **Key type: Signing Key**.

Sì, ogni chiave va caricata due volte (Authentication + Signing): GitHub le tratta come ruoli separati anche se il materiale crittografico è identico.

A fine procedura su GitHub avrai 4 entries: 2 Authentication Keys + 2 Signing Keys.

### 8.2 Configura Git per firmare con SSH

**Su Windows (PowerShell):**

```powershell
git config --global gpg.format ssh
git config --global user.signingkey "$env:USERPROFILE\.ssh\id_ed25519.pub"
git config --global commit.gpgsign true
git config --global tag.gpgsign true
```

**Su WSL (bash):**

```bash
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global commit.gpgsign true
git config --global tag.gpgsign true
```

Dettaglio delle opzioni:

- **`gpg.format ssh`:** dice a Git di usare SSH (non GPG) come formato di firma.
- **`user.signingkey`:** punta al file della chiave **pubblica**.
- **`commit.gpgsign true`:** firma automatica di ogni commit.
- **`tag.gpgsign true`:** firma automatica di ogni tag annotato.

> **Nota:** non impostare `push.gpgSign true` salvo che tu sappia di volere i *signed push* (feature server-side diversa, raramente necessaria — il badge "Verified" non dipende da questa opzione).

### 8.3 Configura il file `allowed_signers` per la verifica locale

Serve a Git per verificare le firme nei tuoi log locali con `git log --show-signature`.

**Su Windows (PowerShell):**

```powershell
$signer = "$(git config user.email) namespaces=`"git`" $(Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub)"
$signer | Out-File -FilePath "$env:USERPROFILE\.ssh\allowed_signers" -Encoding ascii
git config --global gpg.ssh.allowedSignersFile "$env:USERPROFILE/.ssh/allowed_signers"
```

**Su WSL (bash):**

```bash
echo "$(git config user.email) namespaces=\"git\" $(cat ~/.ssh/id_ed25519.pub)" > ~/.ssh/allowed_signers
git config --global gpg.ssh.allowedSignersFile ~/.ssh/allowed_signers
```

### 8.4 Forza l'uso dell'OpenSSH di sistema (solo Windows)

Su Windows, Git Bash include una sua versione di SSH che a volte non riesce a parlare con `ssh-agent` di Windows. Forza Git a usare l'OpenSSH nativo:

```powershell
git config --global core.sshCommand "C:/Windows/System32/OpenSSH/ssh.exe"
```

Su WSL non serve: usa già l'OpenSSH di sistema.

---

## 9. Configurazioni Git "quality of life"

Queste opzioni cambiano davvero la qualità della vita quotidiana con Git. Lanciale tutte una volta e dimenticatene. **Da fare in entrambi gli ambienti.**

**Su Windows e WSL (gli stessi comandi):**

```bash
git config --global init.defaultBranch main
git config --global push.autoSetupRemote true
git config --global fetch.prune true
git config --global pull.rebase true
git config --global rebase.autoStash true
git config --global rerere.enabled true
git config --global merge.conflictstyle zdiff3
git config --global help.autocorrect 20
git config --global column.ui auto
git config --global branch.sort -committerdate
git config --global tag.sort version:refname
git config --global diff.algorithm histogram
git config --global diff.colorMoved default
```

L'unica differenza tra Windows e WSL:

**Su Windows:**
```powershell
git config --global core.autocrlf true
git config --global core.editor "code --wait"
```

**Su WSL:**
```bash
git config --global core.autocrlf input
git config --global core.editor "code --wait"
```

Spiegazione una per una:

**`init.defaultBranch main`** — i nuovi repo creati con `git init` partono con `main` invece di `master`. *Perché:* è lo standard moderno; coerenza con GitHub.

**`push.autoSetupRemote true`** — al primo push di un branch nuovo, Git lo collega automaticamente al remoto con lo stesso nome. *Perché:* elimina il fastidioso `fatal: The current branch has no upstream branch...`.

**`fetch.prune true`** — durante un `git fetch`/`pull`, rimuove dai riferimenti locali i branch cancellati sul remoto. *Perché:* senza questa opzione, dopo qualche mese hai una giungla di branch fantasma.

**`pull.rebase true`** — `git pull` ribasa invece di fare merge. *Perché:* la storia rimane lineare e leggibile; eviti i commit `Merge branch 'main' of...` che inquinano il log.

**`rebase.autoStash true`** — Git stasha automaticamente le modifiche non committate prima di un rebase e le ripristina dopo. *Perché:* senza questo, ogni rebase con working tree sporco si ferma con un errore.

**`rerere.enabled true`** — "Reuse Recorded Resolution": Git memorizza come risolvi un conflitto e la prossima volta lo risolve da solo. *Perché:* utilissimo per chi fa rebase di branch lunghi o lavora con merge ricorrenti.

**`merge.conflictstyle zdiff3`** — durante i conflitti mostra anche il "common ancestor". *Perché:* capire da dove sono partite le due versioni rende molto più chiaro come riconciliarle. Disponibile da Git 2.35+.

**`help.autocorrect 20`** — se sbagli a scrivere un comando, Git lo corregge automaticamente dopo 2 secondi (20 decimi). *Perché:* `git ststaus` diventa `git status` senza dover ribattere.

**`column.ui auto`** — output di `git branch` e simili in colonne quando ci sono molti elementi. *Perché:* leggibilità.

**`branch.sort -committerdate`** — `git branch` mostra i branch ordinati dal più recentemente modificato. *Perché:* trovi al volo l'ultimo branch su cui stavi lavorando.

**`tag.sort version:refname`** — `git tag` mostra i tag ordinati come versioni (1.10 > 1.9). *Perché:* default alfabetico mette `1.10` prima di `1.2`. Insensato per versioni semver.

**`diff.algorithm histogram`** — algoritmo di diff più intelligente del default `myers`. *Perché:* produce diff più leggibili nei refactoring.

**`diff.colorMoved default`** — evidenzia con colore diverso le righe spostate (non aggiunte/rimosse, solo spostate). *Perché:* riconoscere a colpo d'occhio cosa è stato realmente cambiato.

**`core.autocrlf true` (Windows) / `input` (WSL)** — gestione dei line ending. Su Windows converte CRLF↔LF al checkout/commit; su WSL lascia passare LF e converte CRLF in LF al commit. *Perché:* evita conflitti tra membri di team su sistemi operativi diversi.

**`core.editor "code --wait"`** — apre VS Code per messaggi di commit lunghi, conflitti, rebase interattivi. *Perché:* il default è Vim, infernale se non lo conosci.

---

## 10. Diff e merge tool con VS Code

VS Code ha una UI eccellente per visualizzare diff e risolvere conflitti, molto migliore della command line.

**Su Windows e WSL (stessi comandi):**

```bash
git config --global diff.tool vscode
git config --global difftool.vscode.cmd 'code --wait --diff $LOCAL $REMOTE'
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait --merge $REMOTE $LOCAL $BASE $MERGED'
git config --global mergetool.keepBackup false
```

- **`diff.tool` / `merge.tool`:** dichiarano lo strumento da usare quando lanci `git difftool` o `git mergetool`.
- **`mergetool.keepBackup false`:** non lasciare in giro file `.orig` dopo i merge. *Perché:* sono inutili (Git stesso è il backup) e finiscono per essere committati per errore.

---

## 11. Alias Git utili

Gli alias sono comandi personalizzati. Imposta gli stessi in entrambi gli ambienti.

```bash
git config --global alias.st "status -sb"
git config --global alias.co "checkout"
git config --global alias.cm "commit -m"
git config --global alias.amend "commit --amend --no-edit"
git config --global alias.last "log -1 HEAD --stat"
git config --global alias.unstage "reset HEAD --"
git config --global alias.lg "log --graph --abbrev-commit --decorate --format=format:'%C(bold blue)%h%C(reset) - %C(bold green)(%ar)%C(reset) %C(white)%s%C(reset) %C(dim white)- %an%C(reset)%C(bold yellow)%d%C(reset)' --all"
git config --global alias.brs "!f() { git branch --sort=-committerdate --color=always | grep -E '^(\\*| ) (main|remotes/origin/main)$'; git branch --sort=-committerdate --color=always | grep -E '^(\\*| ) (develop|remotes/origin/develop)$'; git branch --sort=-committerdate --color=always | grep -vE ' (main|develop)$'; }; f"
git config --global alias.cleanup "!git branch --merged | grep -vE '^\\*|main$|develop$' | xargs -n 1 git branch -d"
git config --global alias.uncommit "reset --soft HEAD~1"
git config --global alias.wip "!git add -A && git commit -m 'WIP' --no-verify"
```

Cosa fanno:

- **`git st`** — `status` in formato compatto con info su branch upstream.
- **`git co`** — alias breve per `checkout`.
- **`git cm "messaggio"`** — `commit -m`.
- **`git amend`** — aggiunge le modifiche staged all'ultimo commit senza modificare il messaggio.
- **`git last`** — mostra l'ultimo commit con il diff dei file toccati.
- **`git unstage <file>`** — toglie un file dallo staging.
- **`git lg`** — log grafico colorato di tutti i branch. **Il comando più utile in assoluto.**
- **`git brs`** — lista branch con `main` e `develop` in cima, poi gli altri per data.
- **`git cleanup`** — elimina i branch locali già mergiati su main/develop. Pulizia rapida dopo PR mergiate.
- **`git uncommit`** — annulla l'ultimo commit lasciando le modifiche staged. Utile quando hai committato troppo presto.
- **`git wip`** — commit veloce di "lavoro in corso" saltando gli hook (per sapere dove eri quando devi riprendere).

---

## 12. Gitignore globale

Esistono file che non dovrebbero **mai** finire in nessun repository: file di sistema operativo, file di editor, cache. Invece di ricordarseli per ogni progetto, configura un `.gitignore` globale valido per tutti.

**Su Windows (PowerShell):**

```powershell
git config --global core.excludesfile "$env:USERPROFILE\.gitignore_global"

@"
# Sistema operativo - Windows
Thumbs.db
desktop.ini
`$RECYCLE.BIN/

# Sistema operativo - macOS (utile se ricevi repo da utenti Mac)
.DS_Store
._*

# Editor e IDE
.vscode/
.idea/
*.swp
*.swo
*~
.history/

# Log e cache
*.log
.cache/

# File di configurazione locale
.env.local
"@ | Out-File -FilePath "$env:USERPROFILE\.gitignore_global" -Encoding utf8
```

**Su WSL (bash):**

```bash
git config --global core.excludesfile ~/.gitignore_global

cat > ~/.gitignore_global << 'EOF'
# Sistema operativo - Linux
*~
.directory
.Trash-*

# Sistema operativo - macOS (utile se ricevi repo da utenti Mac)
.DS_Store
._*

# Sistema operativo - Windows (utile se ricevi repo Windows)
Thumbs.db
desktop.ini

# Editor e IDE
.vscode/
.idea/
*.swp
*.swo
.history/

# Log e cache
*.log
.cache/

# File di configurazione locale
.env.local
EOF
```

*Perché:* se non lo fai, prima o poi qualcuno scaricherà il tuo repo e troverà un `.DS_Store` o un `.idea/` committato. È sciatto e si vede.

---

## 13. Credential helper per HTTPS

Anche se userai SSH per i tuoi repo, prima o poi ne cloni uno via HTTPS (per esempio uno aziendale dietro proxy). Configura il credential manager per non dover digitare la password ogni volta.

**Su Windows:**

```powershell
git config --global credential.helper manager
```

**Su WSL:** consigliato usare **Git Credential Manager** di Windows da WSL. Aggiungi:

```bash
git config --global credential.helper "/mnt/c/Program\\ Files/Git/mingw64/bin/git-credential-manager.exe"
```

In alternativa, installazione diretta in WSL:

```bash
git config --global credential.helper store
```

`store` salva le credenziali in `~/.git-credentials` in chiaro: usa solo se sai che il tuo home WSL è accessibile solo da te. Per ambienti condivisi, preferisci sempre Git Credential Manager.

*Perché:* `manager` si integra con Windows Credential Store e gestisce anche il flusso OAuth di GitHub via browser. Una volta autenticato, tutto cached in modo sicuro. Riusare quello di Windows da WSL è la scelta più ergonomica.

---

## 14. Configurazione della shell

### 14.1 Windows: PowerShell con posh-git

`posh-git` aggiunge a PowerShell autocompletamento per Git e mostra lo stato del repo nel prompt.

**PowerShell come amministratore:**

```powershell
Install-Module posh-git -Scope CurrentUser -Force
```

**PowerShell normale:**

```powershell
if (!(Test-Path $PROFILE)) { New-Item -Path $PROFILE -ItemType File -Force }
Add-Content -Path $PROFILE -Value "Import-Module posh-git"
. $PROFILE
```

### 14.2 WSL: bash completion + prompt utile

Bash su Ubuntu ha già il completamento Git installato. Migliora il prompt con info sul branch corrente. Aggiungi a `~/.bashrc`:

```bash
# Mostra il branch Git corrente nel prompt
parse_git_branch() {
    git branch 2>/dev/null | sed -n -e 's/^\* \(.*\)/ (\1)/p'
}

PS1='\[\e[32m\]\u@\h\[\e[m\]:\[\e[34m\]\w\[\e[m\]\[\e[33m\]$(parse_git_branch)\[\e[m\]\$ '
```

Ricarica:

```bash
source ~/.bashrc
```

### 14.3 Alternativa professionale: Zsh + Oh My Zsh + Powerlevel10k

Per un'esperienza terminale di livello superiore in WSL, molti professionisti passano a Zsh:

```bash
sudo apt install -y zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

Poi installa Powerlevel10k come tema (mostra branch Git, stato pull/push, durata comandi, virtualenv attivo, ecc.). *Perché:* migliore esperienza visiva, autocompletamento più potente, plugin per Git, Docker, Node, Python integrati.

Optional ma raccomandato per chi passa molto tempo nel terminale.

---

## 15. GitHub CLI

`gh` è il client ufficiale GitHub da terminale. Permette di creare repo, aprire pull request, gestire issue senza aprire il browser.

**Da entrambi gli ambienti:**

```bash
gh auth login
```

Procedura:
- **GitHub.com**
- **SSH** come protocollo (la chiave dell'ambiente corrente).
- Autenticazione via browser.

Comandi essenziali:

```bash
gh repo create mio-progetto --private --source=. --remote=origin --push
gh repo clone owner/repo
gh pr create --base main --head feature/login --title "Aggiunta login" --body "..."
gh pr list
gh pr checkout 42
gh pr review 42 --approve
gh issue create --title "Bug nel login" --body "..."
gh issue list --state open
gh release create v1.0.0 --notes "Prima release"
```

Configurazioni utili:

```bash
gh config set editor "code --wait"
gh config set git_protocol ssh
gh config set prompt enabled
```

*Perché `gh`:* per chi vive nel terminale, è molto più veloce del browser. Inoltre è scriptable: puoi automatizzare workflow (creare PR da CI, generare changelog, ecc.).

### Estensioni utili

```bash
gh extension install github/gh-copilot   # GitHub Copilot CLI (se hai licenza)
gh extension install dlvhdr/gh-dash      # Dashboard interattiva di PR e issue
```

---

## 16. Integrazione con VS Code

### 16.1 Estensione Remote - WSL (essenziale)

Su VS Code Windows, installa l'estensione **WSL** (id: `ms-vscode-remote.remote-wsl`).

Apri un progetto in WSL:

```bash
cd ~/progetti/mio-progetto
code .
```

VS Code Windows si avvia ma il backend (terminale, debugger, estensioni) gira **dentro** Ubuntu. *Perché:* hai la UI Windows con la potenza dello stack Linux. Niente performance penalty, line ending corretti, comandi shell nativi.

### 16.2 Estensioni Git essenziali

Installa in VS Code (sezione Extensions):

- **GitLens** — supercharge per Git in VS Code: blame inline, history, lens, compare. Praticamente obbligatorio.
- **GitHub Pull Requests and Issues** — estensione ufficiale GitHub: gestisci PR, review, issue direttamente dall'editor.
- **Git Graph** — visualizzatore del grafo dei commit nel pannello SCM.
- **Conventional Commits** — autocompletamento per messaggi di commit con format conventional (`feat:`, `fix:`, ecc.).

### 16.3 Configurazioni VS Code consigliate

Apri **File → Preferences → Settings** e cerca:

- `git.autofetch`: **true** — fa fetch automatico ogni 3 minuti, così vedi sempre se ci sono novità sul remoto.
- `git.confirmSync`: **false** — non chiedere conferma ogni volta che pulli/pushi.
- `git.enableSmartCommit`: **true** — se non hai nulla in staged, committa tutti i modificati.
- `git.suggestSmartCommit`: **false** — disattiva il prompt di suggerimento.
- `editor.formatOnSave`: **true** — formatta il codice al salvataggio (richiede formatter installato).
- `files.trimTrailingWhitespace`: **true** — rimuove spazi a fine riga al salvataggio.
- `files.insertFinalNewline`: **true** — aggiunge newline finale ai file (standard POSIX).

### 16.4 Sync delle settings

In VS Code, in basso a sinistra, attiva **Settings Sync** loggandoti con il tuo account GitHub. *Perché:* settings, estensioni, scorciatoie, snippet vengono sincronizzati tra tutti i tuoi PC.

---

## 17. Verifica finale

Controlla che tutto sia configurato correttamente in entrambi gli ambienti.

```bash
git config --global --list
ssh -T git@github.com
gh auth status
```

### Test del commit firmato (esegui nell'ambiente che usi di più, tipicamente WSL)

```bash
mkdir test-github && cd test-github
git init
echo "# Test" > README.md
git add .
git commit -m "Primo commit firmato"
git log --show-signature
```

L'ultima riga deve mostrare `Good "git" signature for tua@email`.

### Test del push

```bash
gh repo create test-github --private --source=. --remote=origin --push
```

Vai su GitHub, apri il repo: il commit deve avere un badge verde **Verified** accanto.

Se il badge è verde, hai finito: tutta la pipeline (Git locale → SSH key → SSH signing → GitHub) funziona.

Cleanup:

```bash
cd ..
gh repo delete test-github --yes
rm -rf test-github
```

---

## 18. Best practice per i tuoi repository

Configurazione fatta. Qualche regola d'oro per i repo che creerai d'ora in poi.

### File essenziali

**README.md** sempre presente, anche minimale: nome del progetto, cosa fa, come si installa, come si usa, come contribuire. Senza README, un repo è invisibile a chiunque non sia te.

**LICENSE** sempre presente nei repo pubblici. Senza licenza, legalmente nessuno può usare il tuo codice. Per progetti personali la **MIT** è la scelta più comune e permissiva. Per progetti che vuoi proteggere dalla rivendita commerciale, **AGPL-3.0** o **GPL-3.0**.

**`.gitignore`** specifico al linguaggio/framework. [github.com/github/gitignore](https://github.com/github/gitignore) ha template pronti per ogni tecnologia.

**`.gitattributes`** per progetti cross-platform: forza LF su file critici (`*.sh`, `Dockerfile`, `Makefile`).

```
* text=auto eol=lf
*.sh text eol=lf
*.bat text eol=crlf
*.png binary
```

**`CONTRIBUTING.md`** per repo open source: come fare PR, come scrivere commit, come testare.

**`SECURITY.md`** per dichiarare come segnalare vulnerabilità in modo responsabile (privato, non come issue pubblica).

### Configurazione del repository su GitHub

Nei repo importanti, vai su **Settings**:

- **General → Default branch:** `main`.
- **General → Pull Requests → Allow merge commits/squash/rebase:** scegli la strategia. La più pulita per progetti piccoli è **solo Squash and merge** (PR mergiata = 1 commit su main).
- **General → Pull Requests → Automatically delete head branches:** **attiva**. Dopo merge, il branch della PR viene cancellato automaticamente.
- **Branches → Branch protection rules** per `main`:
    - Require pull request before merging
    - Require approvals (almeno 1 reviewer)
    - Require status checks to pass (CI verde)
    - Require conversation resolution before merging
    - Require signed commits
    - Do not allow bypassing the above settings (anche per admin)
- **Code security → Dependabot:** attiva alerts e security updates. Riceverai PR automatiche per aggiornare dipendenze vulnerabili.
- **Code security → Code scanning** (CodeQL): attiva per repo che ne valgono la pena.
- **Code security → Secret scanning:** attiva. GitHub scansiona i commit per chiavi API/token e ti avverte se ne lasci una.

### Convenzioni per i commit

**Conventional Commits** ([conventionalcommits.org](https://www.conventionalcommits.org)):

```
feat: aggiunto endpoint per il login
fix: corretto crash quando il token è scaduto
docs: aggiornato README con istruzioni di installazione
refactor: estratto modulo di validazione
test: aggiunti test per il flusso di registrazione
chore: aggiornata versione di node
```

Tipi: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `chore`, `ci`, `build`.

*Perché:* permettono di generare changelog automatici (con `release-please`, `semantic-release`, ecc.) e rendono la storia leggibile a colpo d'occhio. Standard nel mondo professionale.

### Branch strategy

**Anche per progetti personali:** mai lavorare direttamente su `main`.

- Crea un branch per ogni feature/fix: `feat/nome-feature`, `fix/nome-bug`.
- Lavora, committa, pusha sul branch.
- Apri PR (anche su repo solo tuoi: ti permette di rivedere le tue stesse modifiche prima di mergiare).
- Mergia con squash o rebase.
- Cancella il branch.

*Perché:* ti abitui al workflow professionale dei team, e in caso di errore puoi sempre buttare via il branch senza compromettere `main`.

### Sicurezza dei segreti

**Mai committare API key, password, token, file `.env`.** Anche se cancelli il commit dopo, **resta nella history per sempre** finché non riscrivi tutto (con `git filter-repo` o BFG). Se ti capita, considera quel segreto compromesso e ruotalo subito.

Strumenti consigliati:

- **`gitleaks`** o **`trufflehog`** in pre-commit hook: scansionano i file in commit per pattern di credenziali.
- **`git-secrets`** di AWS: simile, focalizzato su credenziali AWS.

Installa pre-commit hooks:

```bash
sudo apt install -y pre-commit  # in WSL
# o pip install pre-commit
```

E in ogni repo aggiungi `.pre-commit-config.yaml` con i check rilevanti.

### Dotfiles repository

Crea un repo (privato o pubblico) chiamato `dotfiles` con dentro le tue configurazioni: `.bashrc`, `.zshrc`, `.gitconfig`, settings VS Code, config tmux. *Perché:* su un nuovo PC cloni il repo, lanci uno script di setup ed sei operativo in 10 minuti invece che in 4 ore.

### Stargazing strategico

Metti la stella ai progetti che usi davvero. *Perché:* diventa la tua personal library — invece di chiedere "come si chiamava quella libreria che facevo X?" cerchi nelle tue stelle. Inoltre supporta i maintainer.

---

## 19. Cheat sheet completo

Tutti i comandi `git config --global` di questa guida in un unico blocco. Eseguili **una volta in Windows** e **una volta in WSL** dopo aver installato Git e creato le chiavi SSH.

### Identità
```bash
git config --global user.name "Mario Rossi"
git config --global user.email "12345+mariorossi@users.noreply.github.com"
```

### Quality of life (uguali in entrambi gli ambienti)
```bash
git config --global init.defaultBranch main
git config --global push.autoSetupRemote true
git config --global fetch.prune true
git config --global pull.rebase true
git config --global rebase.autoStash true
git config --global rerere.enabled true
git config --global merge.conflictstyle zdiff3
git config --global help.autocorrect 20
git config --global column.ui auto
git config --global branch.sort -committerdate
git config --global tag.sort version:refname
git config --global diff.algorithm histogram
git config --global diff.colorMoved default
git config --global core.editor "code --wait"
```

### Specifico Windows
```powershell
git config --global core.autocrlf true
git config --global core.sshCommand "C:/Windows/System32/OpenSSH/ssh.exe"
git config --global credential.helper manager
git config --global core.excludesfile "$env:USERPROFILE\.gitignore_global"
git config --global user.signingkey "$env:USERPROFILE\.ssh\id_ed25519.pub"
git config --global gpg.ssh.allowedSignersFile "$env:USERPROFILE/.ssh/allowed_signers"
```

### Specifico WSL
```bash
git config --global core.autocrlf input
git config --global credential.helper "/mnt/c/Program\\ Files/Git/mingw64/bin/git-credential-manager.exe"
git config --global core.excludesfile ~/.gitignore_global
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global gpg.ssh.allowedSignersFile ~/.ssh/allowed_signers
```

### SSH signing (uguali)
```bash
git config --global gpg.format ssh
git config --global commit.gpgsign true
git config --global tag.gpgsign true
```

### Diff/Merge tool (uguali)
```bash
git config --global diff.tool vscode
git config --global difftool.vscode.cmd 'code --wait --diff $LOCAL $REMOTE'
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait --merge $REMOTE $LOCAL $BASE $MERGED'
git config --global mergetool.keepBackup false
```

### Alias (uguali)
```bash
git config --global alias.st "status -sb"
git config --global alias.co "checkout"
git config --global alias.cm "commit -m"
git config --global alias.amend "commit --amend --no-edit"
git config --global alias.last "log -1 HEAD --stat"
git config --global alias.unstage "reset HEAD --"
git config --global alias.uncommit "reset --soft HEAD~1"
git config --global alias.wip "!git add -A && git commit -m 'WIP' --no-verify"
git config --global alias.cleanup "!git branch --merged | grep -vE '^\\*|main$|develop$' | xargs -n 1 git branch -d"
git config --global alias.lg "log --graph --abbrev-commit --decorate --format=format:'%C(bold blue)%h%C(reset) - %C(bold green)(%ar)%C(reset) %C(white)%s%C(reset) %C(dim white)- %an%C(reset)%C(bold yellow)%d%C(reset)' --all"
git config --global alias.brs "!f() { git branch --sort=-committerdate --color=always | grep -E '^(\\*| ) (main|remotes/origin/main)$'; git branch --sort=-committerdate --color=always | grep -E '^(\\*| ) (develop|remotes/origin/develop)$'; git branch --sort=-committerdate --color=always | grep -vE ' (main|develop)$'; }; f"
```

### GitHub CLI
```bash
gh auth login
gh config set editor "code --wait"
gh config set git_protocol ssh
```

---

## Riepilogo: cosa hai ottenuto seguendo questa guida

- Account GitHub sicuro con 2FA, recovery codes salvati, passkey, email privata.
- Profilo professionale con README profilo e pinned repos.
- WSL2 Ubuntu installato come ambiente di sviluppo principale.
- Git configurato con identità coerente in entrambi gli ambienti.
- Due chiavi SSH separate (Windows + WSL) sia per autenticazione che per signing.
- Tutti i tuoi commit avranno il badge **Verified** verde.
- Configurazioni Git ottimali: rebase pulito, prune automatico, rerere, conflict style migliore, diff intelligente.
- Alias che velocizzano il lavoro quotidiano.
- Gitignore globale che evita di committare spazzatura.
- Shell con autocompletamento Git e branch nel prompt.
- VS Code integrato con WSL e GitHub.
- GitHub CLI configurato per gestire tutto da terminale.
- Best practice per repository, branch protection, conventional commits, secret scanning.

Sei configurato come un senior developer. Il setup va rivisitato una volta all'anno per restare allineato con eventuali nuove best practice.

---

*Guida aggiornata a maggio 2026.*
