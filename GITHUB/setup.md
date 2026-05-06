# Configurazione Professionale di GitHub — Guida Completa

> Guida completa per configurare GitHub e Git da zero in modo professionale, sicuro ed efficiente su Windows. Ogni passo è accompagnato dalla spiegazione del *perché* serve.

---

## Indice

1. [Creazione dell'account GitHub](#1-creazione-dellaccount-github)
2. [Sicurezza dell'account (priorità massima)](#2-sicurezza-dellaccount-priorità-massima)
3. [Profilo pubblico professionale](#3-profilo-pubblico-professionale)
4. [Installazione del software](#4-installazione-del-software)
5. [Identità Git locale](#5-identità-git-locale)
6. [Chiave SSH per autenticazione](#6-chiave-ssh-per-autenticazione)
7. [Commit verificati con SSH signing](#7-commit-verificati-con-ssh-signing)
8. [Configurazioni Git "quality of life"](#8-configurazioni-git-quality-of-life)
9. [Diff e merge tool con VS Code](#9-diff-e-merge-tool-con-vs-code)
10. [Alias Git utili](#10-alias-git-utili)
11. [Gitignore globale](#11-gitignore-globale)
12. [Credential helper per HTTPS](#12-credential-helper-per-https)
13. [PowerShell con posh-git](#13-powershell-con-posh-git)
14. [GitHub CLI](#14-github-cli)
15. [Verifica finale](#15-verifica-finale)
16. [Best practice per i tuoi repository](#16-best-practice-per-i-tuoi-repository)

---

## 1. Creazione dell'account GitHub

Vai su [github.com](https://github.com) e clicca **Sign up**.

**Email da usare.** Usa un indirizzo email *personale* a cui avrai accesso a vita, non un'email scolastica o aziendale. *Perché:* l'email scolastica viene disattivata dopo la laurea e quella aziendale alla fine del rapporto di lavoro; perdere l'accesso all'email significa rischiare di perdere l'account GitHub e tutti i tuoi contributi pubblici. Potrai sempre collegare un account Google in un secondo momento per il login rapido.

**Scelta dello username.** Lo username GitHub è il tuo biglietto da visita per recruiter, colleghi e contributori open source: appare in ogni URL dei tuoi repo (`github.com/tuousername/...`) e in ogni commit. Regole:

- Solo nome e cognome (es. `mariorossi`, `mario-rossi`, `mrossi`).
- Niente numeri casuali, niente nickname da gaming, niente date di nascita.
- Ammessi: lettere, numeri e trattini singoli. Vietati spazi e caratteri speciali.
- Massimo 39 caratteri.

*Perché:* uno username serio comunica professionalità. `xXmario99Xx` non viene mai assunto.

**Verifica l'email** cliccando il link di conferma. Senza verifica email non puoi né pushare né essere riconosciuto come autore dei commit.

---

## 2. Sicurezza dell'account (priorità massima)

GitHub contiene il tuo lavoro, le tue credenziali (token, API key nei `.env` dimenticati) e il tuo nome. Un account compromesso è un disastro. Fai questi passi *prima* di iniziare a lavorare.

### 2.1 Attiva l'autenticazione a due fattori (2FA)

Vai su **Settings → Password and authentication → Two-factor authentication**.

Scegli **Authenticator app** (NON SMS). *Perché:* gli SMS sono vulnerabili al SIM swapping; un'app TOTP come Google Authenticator, Microsoft Authenticator, Authy o 1Password è offline e molto più sicura.

Scansiona il QR code con la tua app authenticator e inserisci il codice a 6 cifre per confermare.

### 2.2 Salva i recovery codes (passo più importante della guida)

Dopo l'attivazione del 2FA, GitHub ti mostra **16 recovery codes**. Salvali subito.

**Dove salvarli:**

- In un password manager (Bitwarden, 1Password, KeePass).
- Stampati e conservati in un posto fisico sicuro.
- **Mai** in chiaro su Desktop, in note di Telegram a te stesso, o in email.

*Perché:* se perdi/rompi/resetti il telefono con l'app authenticator e non hai i recovery codes, **perdi l'account in modo definitivo**. GitHub non ha un servizio clienti che ti riapre l'account: la chiave sei tu. Questa è la causa numero uno di account persi.

### 2.3 Aggiungi una passkey (opzionale ma consigliato)

In **Settings → Password and authentication → Passkeys** aggiungi una passkey (Windows Hello, FaceID, YubiKey). Permette login senza password e resistente al phishing.

### 2.4 Email di backup

In **Settings → Emails** aggiungi una seconda email come backup. Se perdi accesso alla principale, hai una via di recupero in più.

### 2.5 Privacy delle email nei commit

In **Settings → Emails** attiva **Keep my email addresses private** e **Block command line pushes that expose my email**.

GitHub ti darà un'email "noreply" del tipo `12345+username@users.noreply.github.com` da usare nei commit. *Perché:* ogni commit pubblico contiene l'email dell'autore in chiaro per sempre; usando l'email noreply eviti spam, harvesting da bot e fughe accidentali della tua email personale.

---

## 3. Profilo pubblico professionale

Vai su **Settings → Public profile**.

- **Nome:** il tuo nome reale (es. "Mario Rossi"). Recruiters cercano persone, non handle.
- **Foto:** una foto professionale, ben illuminata, viso visibile. Stesso standard di LinkedIn.
- **Bio:** una riga, cosa fai e a cosa ti interessa lavorare.
- **Location, sito, social:** opzionali, ma se li metti tienili aggiornati.

### Profile README (carta da visita pubblica)

Crea un repository con **lo stesso identico nome del tuo username** (es. se ti chiami `mariorossi`, il repo si chiama `mariorossi`). Aggiungi un file `README.md` al suo interno: GitHub lo mostrerà automaticamente in cima alla tua pagina profilo.

*Perché:* è la prima cosa che vede chi visita il tuo profilo. Mettici una breve presentazione, le tecnologie su cui lavori e qualche progetto che metti in evidenza.

---

## 4. Installazione del software

Apri **PowerShell come amministratore** e installa tutto via `winget`:

```powershell
winget install --id Git.Git -e
winget install --id Microsoft.VisualStudioCode -e
winget install --id GitHub.cli -e
```

*Perché winget:* è il package manager ufficiale di Windows, le installazioni sono pulite e gli aggiornamenti gestibili da terminale (`winget upgrade --all`). Niente caccia all'installer su siti random.

Chiudi e riapri PowerShell, poi verifica:

```powershell
git --version
code --version
gh --version
```

Se uno dei tre dà errore, riavvia il PC (il `PATH` a volte si aggiorna solo dopo un reboot).

---

## 5. Identità Git locale

Git deve sapere chi sei: ogni commit viene firmato con nome ed email. Imposta i valori globalmente (validi per tutti i repository):

```powershell
git config --global user.name "Mario Rossi"
git config --global user.email "12345+mariorossi@users.noreply.github.com"
```

*Perché:* nome ed email appaiono in ogni commit, in `git log`, su GitHub e nelle storie dei progetti open source a cui contribuirai. L'email **deve corrispondere** a una verificata sul tuo account GitHub, altrimenti i commit non vengono associati al tuo profilo (rimangono "anonimi" pur essendo firmati da te).

Se hai attivato la privacy email del passo 2.5, usa l'email noreply che trovi in **Settings → Emails** sotto "Keep my email addresses private". Lì c'è scritta esattamente.

---

## 6. Chiave SSH per autenticazione

Le credenziali SSH sostituiscono la password per push, pull e clone. Sono più sicure e non si digitano ad ogni operazione.

### 6.1 Genera la chiave

In PowerShell (utente normale, non admin):

```powershell
ssh-keygen -t ed25519 -C "12345+mariorossi@users.noreply.github.com"
```

- **`-t ed25519`:** algoritmo moderno, più veloce e più sicuro di RSA. Da preferire sempre, salvo lavoro su sistemi legacy che supportano solo RSA.
- **`-C "..."`:** etichetta che apparirà nel commento della chiave (l'email è la convenzione).

Quando ti chiede dove salvare, premi Invio (default: `~/.ssh/id_ed25519`).

**Quando chiede la passphrase, INSERISCINE UNA.** Non lasciarla vuota.

*Perché:* la chiave privata è un file sul tuo disco. Se ti rubano il PC, o un malware ne fa l'esfiltrazione, senza passphrase chi ha il file ha accesso completo a tutti i tuoi repo (incluso il push di codice malevolo a tuo nome). Con la passphrase serve anche quella per usarla. Salvala nel password manager.

### 6.2 Configura ssh-agent (per non digitare la passphrase ogni volta)

Apri **PowerShell come amministratore**:

```powershell
Get-Service -Name ssh-agent | Set-Service -StartupType Automatic
Start-Service ssh-agent
```

*Perché:* l'`ssh-agent` è un servizio che tiene la chiave decifrata in memoria per la durata della sessione, così digiti la passphrase una volta sola.

Poi torna a PowerShell utente normale e aggiungi la chiave all'agent:

```powershell
ssh-add $env:USERPROFILE\.ssh\id_ed25519
```

Inserisci la passphrase. Da ora ssh-agent gestisce tutto.

### 6.3 Aggiungi la chiave pubblica a GitHub

Copia la chiave **pubblica** negli appunti:

```powershell
Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub | Set-Clipboard
```

Attenzione: copia il `.pub` (pubblica). Quella **senza** estensione è la chiave privata e non deve mai uscire dal tuo PC.

Su GitHub: **Settings → SSH and GPG keys → New SSH key**.

- **Title:** un nome riconoscibile (es. "Laptop personale Windows").
- **Key type:** lascia `Authentication Key`.
- **Key:** incolla.

### 6.4 Verifica la connessione

```powershell
ssh -T git@github.com
```

Risposta attesa: `Hi mariorossi! You've successfully authenticated, but GitHub does not provide shell access.` Il messaggio "but GitHub does not provide shell access" è normale e significa che ha funzionato.

---

## 7. Commit verificati con SSH signing

I "Verified commits" mostrano un badge verde su GitHub e garantiscono che il commit è stato firmato con la tua chiave. Senza firma, chiunque può configurare Git con il tuo nome ed email e committare a tuo nome (l'autore di un commit Git non è autenticato di default).

Useremo la **stessa chiave SSH** del passo precedente per firmare anche i commit. Non serve creare GPG: SSH signing è supportato da Git 2.34+ ed è la pratica moderna consigliata.

### 7.1 Aggiungi la chiave anche come Signing Key su GitHub

Su GitHub: **Settings → SSH and GPG keys → New SSH key**. Incolla la **stessa identica chiave pubblica** del passo 6.3, ma stavolta seleziona **Key type: Signing Key**.

Sì, la stessa chiave va caricata due volte (una come Authentication, una come Signing): GitHub le tratta come ruoli separati anche se il materiale crittografico è identico.

### 7.2 Configura Git per firmare con SSH

```powershell
git config --global gpg.format ssh
git config --global user.signingkey "$env:USERPROFILE\.ssh\id_ed25519.pub"
git config --global commit.gpgsign true
git config --global tag.gpgsign true
```

- **`gpg.format ssh`:** dice a Git di usare SSH (non GPG) come formato di firma.
- **`user.signingkey`:** punta al file della chiave **pubblica**.
- **`commit.gpgsign true`:** firma automatica di ogni commit, senza dover passare `-S` ogni volta.
- **`tag.gpgsign true`:** firma automatica di ogni tag annotato.

> **Nota:** non impostare `push.gpgSign true` salvo che tu sappia di volere i *signed push* (feature server-side diversa, raramente necessaria — il badge "Verified" non dipende da questa opzione).

### 7.3 Configura il file `allowed_signers` per la verifica locale

Serve a Git per verificare le firme nei tuoi log locali (`git log --show-signature`).

```powershell
$signer = "$(git config user.email) namespaces=`"git`" $(Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub)"
$signer | Out-File -FilePath "$env:USERPROFILE\.ssh\allowed_signers" -Encoding ascii
git config --global gpg.ssh.allowedSignersFile "$env:USERPROFILE/.ssh/allowed_signers"
```

### 7.4 Forza Git a usare l'OpenSSH di Windows

Git Bash include una sua versione di SSH che a volte non riesce a parlare con `ssh-agent` di Windows.

```powershell
git config --global core.sshCommand "C:/Windows/System32/OpenSSH/ssh.exe"
```

*Perché:* uniforma il client SSH usato da Git con quello di sistema, evitando errori del tipo "could not load key" quando vai a firmare.

---

## 8. Configurazioni Git "quality of life"

Queste sono le opzioni che cambiano davvero la qualità della vita quotidiana con Git. Lanciale tutte una volta e dimenticatene.

```powershell
git config --global init.defaultBranch main
git config --global push.autoSetupRemote true
git config --global fetch.prune true
git config --global pull.rebase true
git config --global rebase.autoStash true
git config --global rerere.enabled true
git config --global core.autocrlf true
git config --global core.editor "code --wait"
git config --global merge.conflictstyle zdiff3
```

Spiegazione una per una:

**`init.defaultBranch main`** — quando crei un nuovo repo con `git init`, il branch principale si chiama `main`. *Perché:* è lo standard moderno (sostituisce `master`); essere coerenti con GitHub evita confusione.

**`push.autoSetupRemote true`** — al primo push di un branch nuovo, Git lo collega automaticamente al remoto con lo stesso nome. *Perché:* elimina il fastidioso `fatal: The current branch has no upstream branch. To push the current branch and set the remote as upstream, use git push --set-upstream origin nome-branch`. Niente più copia-incolla.

**`fetch.prune true`** — durante un `git fetch` (e quindi anche `pull`), rimuove dai riferimenti locali i branch che sono stati cancellati sul remoto. *Perché:* senza questa opzione, dopo qualche mese hai una giungla di branch fantasma in `git branch -r` che non esistono più.

**`pull.rebase true`** — quando fai `git pull`, invece di creare un *merge commit* di unione, ribasa i tuoi commit locali sopra quelli appena scaricati. *Perché:* la storia rimane lineare e leggibile; eviti i commit `Merge branch 'main' of github.com/...` che inquinano il log.

**`rebase.autoStash true`** — se provi a ribasare con modifiche non committate, Git le mette in stash automaticamente, fa il rebase e le ripristina. *Perché:* senza questo, ogni rebase che parte con un working tree sporco si ferma con un errore e devi gestire lo stash a mano.

**`rerere.enabled true`** — abbreviato di "reuse recorded resolution": Git memorizza come risolvi un conflitto e la prossima volta che lo stesso conflitto si ripresenta lo risolve da solo. *Perché:* utilissimo per chi fa rebase di branch lunghi o lavora con merge ricorrenti.

**`core.autocrlf true`** — converte automaticamente i line ending da CRLF (Windows) a LF (Linux/Mac) al commit, e viceversa al checkout. *Perché:* evita conflitti tra membri di team su sistemi operativi diversi. **Caveat:** per progetti che richiedono LF a tutti i costi (Dockerfile, script bash, makefile) usa un `.gitattributes` per progetto che sovrascriva questa impostazione.

**`core.editor "code --wait"`** — apre VS Code per scrivere messaggi di commit lunghi, risolvere conflitti, modificare rebase interattivi. *Perché:* il default è Vim, che è ottimo se lo sai usare e infernale se no. VS Code è familiare a tutti.

**`merge.conflictstyle zdiff3`** — durante i conflitti, mostra anche il "common ancestor" (la versione di partenza da cui sono partite entrambe le modifiche). *Perché:* capire da dove sono partite le due versioni rende molto più chiaro come riconciliarle. Disponibile da Git 2.35+.

---

## 9. Diff e merge tool con VS Code

VS Code ha una UI eccellente per visualizzare diff e risolvere conflitti, molto migliore della command line.

```powershell
git config --global diff.tool vscode
git config --global difftool.vscode.cmd 'code --wait --diff $LOCAL $REMOTE'
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait --merge $REMOTE $LOCAL $BASE $MERGED'
git config --global mergetool.keepBackup false
```

- **`diff.tool` / `merge.tool`:** dichiarano lo strumento da usare quando lanci `git difftool` o `git mergetool`.
- **`mergetool.keepBackup false`:** non lasciare in giro file `.orig` dopo i merge. *Perché:* sono inutili (Git stesso è il tuo backup) e finiscono per essere committati per errore.

---

## 10. Alias Git utili

Gli alias sono comandi personalizzati. Questi due ti cambieranno la vita.

```powershell
git config --global alias.lg "log --graph --abbrev-commit --decorate --format=format:'%C(bold blue)%h%C(reset) - %C(bold green)(%ar)%C(reset) %C(white)%s%C(reset) %C(dim white)- %an%C(reset)%C(bold yellow)%d%C(reset)' --all"
```

`git lg` — mostra il log come un grafo colorato e leggibile, con tutti i branch in vista. Una volta provato, non torni più al `git log` di default.

```powershell
git config --global alias.brs "!f() { git branch --sort=-committerdate --color=always | grep -E '^(\*| ) (main|remotes/origin/main)$'; git branch --sort=-committerdate --color=always | grep -E '^(\*| ) (develop|remotes/origin/develop)$'; git branch --sort=-committerdate --color=always | grep -vE ' (main|develop)$'; }; f"
```

`git brs` — lista branch con `main` e `develop` sempre in cima, poi tutti gli altri ordinati per ultima modifica. *Perché:* quando hai 30 branch, trovare l'ultimo su cui stavi lavorando è un attimo invece che scrollare a caso.

Aggiungili e altri comandi rapidi:

```powershell
git config --global alias.st "status -sb"
git config --global alias.co "checkout"
git config --global alias.cm "commit -m"
git config --global alias.amend "commit --amend --no-edit"
git config --global alias.last "log -1 HEAD --stat"
git config --global alias.unstage "reset HEAD --"
```

---

## 11. Gitignore globale

Esistono file che non dovrebbero MAI finire in nessun repository, mai: file di sistema operativo, file di editor, cache. Invece di ricordarsi di metterli nel `.gitignore` di ogni progetto, configurane uno globale valido per tutti.

```powershell
git config --global core.excludesfile "$env:USERPROFILE\.gitignore_global"
```

Crea il file:

```powershell
@"
# Sistema operativo
.DS_Store
Thumbs.db
desktop.ini
\$RECYCLE.BIN/

# Editor e IDE
.vscode/
.idea/
*.swp
*.swo
*~

# Log e cache
*.log
.cache/

# File di configurazione locale
.env.local
"@ | Out-File -FilePath "$env:USERPROFILE\.gitignore_global" -Encoding utf8
```

*Perché:* se non lo fai, prima o poi qualcuno scaricherà il tuo repo e troverà un `.DS_Store` o un `.idea/` committato. È sciatto e si vede.

---

## 12. Credential helper per HTTPS

Anche se userai SSH per i tuoi repo, prima o poi cloni un repo via HTTPS (per esempio uno aziendale) o ti capita di pushare via HTTPS. Configura il credential manager di Windows per non dover digitare la password ogni volta.

```powershell
git config --global credential.helper manager
```

*Perché:* `manager` (Git Credential Manager) si integra con Windows Credential Store e gestisce anche il flusso OAuth di GitHub via browser. Una volta autenticato, tutto cached in modo sicuro.

---

## 13. PowerShell con posh-git

`posh-git` aggiunge a PowerShell autocompletamento per i comandi Git e mostra lo stato del repo nel prompt (branch, file modificati, commit ahead/behind).

In **PowerShell come amministratore**:

```powershell
Install-Module posh-git -Scope CurrentUser -Force
```

Poi, da PowerShell normale:

```powershell
if (!(Test-Path $PROFILE)) { New-Item -Path $PROFILE -ItemType File -Force }
Add-Content -Path $PROFILE -Value "Import-Module posh-git"
. $PROFILE
```

*Perché:* `git che<TAB>` completa a `checkout`, `git checkout fea<TAB>` completa al nome del branch. E vedi in tempo reale se hai file modificati senza dover lanciare `git status`.

---

## 14. GitHub CLI

`gh` è il client ufficiale GitHub da terminale. Permette di creare repo, aprire pull request, gestire issue e altro senza aprire il browser.

```powershell
gh auth login
```

Segui la procedura: scegli **GitHub.com**, poi **SSH** come protocollo (la chiave che hai già), poi autenticazione via browser.

Esempi pratici:

```powershell
gh repo create mio-progetto --public --source=. --remote=origin --push
gh pr create --base main --head feature/login --title "Aggiunta login" --body "..."
gh pr list
gh issue list --state open
```

*Perché:* per chi vive nel terminale, è molto più veloce di switchare a Chrome ogni volta che serve aprire una PR.

---

## 15. Verifica finale

Controlla che tutto sia configurato correttamente.

```powershell
git config --global --list
ssh -T git@github.com
gh auth status
```

### Test del commit firmato

```powershell
mkdir test-github
cd test-github
git init
"# Test" | Out-File README.md -Encoding utf8
git add .
git commit -m "Primo commit firmato"
git log --show-signature
```

L'ultima riga deve mostrare qualcosa come `Good "git" signature for tua@email`.

### Test del push

```powershell
gh repo create test-github --private --source=. --remote=origin --push
```

Vai su GitHub, apri il repo: il commit deve avere un badge verde **Verified** accanto.

Se il badge è verde, hai finito: tutta la pipeline (Git locale → SSH key → SSH signing → GitHub) funziona.

Quando hai finito, cancella il repo di test:

```powershell
cd ..
gh repo delete test-github --yes
Remove-Item -Recurse -Force test-github
```

---

## 16. Best practice per i tuoi repository

Configurazione personale fatta. Qualche regola d'oro per i repo che creerai d'ora in poi.

**README.md** sempre presente, anche minimale: nome del progetto, cosa fa, come si installa, come si usa. Senza README, un repo è invisibile a chiunque non sia te.

**LICENSE** sempre presente nei repo pubblici. Senza licenza, legalmente nessuno può usare il tuo codice. Per progetti personali la **MIT** è la scelta più comune e permissiva.

**`.gitignore` per progetto** specifico al linguaggio/framework. [github.com/github/gitignore](https://github.com/github/gitignore) ha template pronti per ogni tecnologia.

**Branch protection rules** sui repo importanti (Settings → Branches): proteggi `main` da push diretti, richiedi PR e review prima del merge. *Perché:* evita di pushare per sbaglio commit non revisionati sul branch di produzione.

**Conventional Commits** per i messaggi: `feat:`, `fix:`, `docs:`, `refactor:`, `chore:`. *Perché:* permettono di generare changelog automatici e rendono la storia leggibile a colpo d'occhio.

**Niente segreti nei commit.** Mai committare API key, password, token, file `.env`. Anche se cancelli il commit dopo, **resta nella storia per sempre** finché non riscrivi tutto. Se ti capita, considera quel segreto compromesso e ruotalo subito.

**Branch feature, non lavoro su `main`.** Anche per progetti personali: ogni feature/fix in un branch dedicato, poi merge tramite PR. Ti abitua al workflow professionale.

---

## Riepilogo dei comandi (cheat sheet)

Tutti i comandi `git config --global` di questa guida in un unico blocco, da eseguire dopo aver installato Git e aver impostato nome/email:

```powershell
# Identità
git config --global user.name "Mario Rossi"
git config --global user.email "12345+mariorossi@users.noreply.github.com"

# Quality of life
git config --global init.defaultBranch main
git config --global push.autoSetupRemote true
git config --global fetch.prune true
git config --global pull.rebase true
git config --global rebase.autoStash true
git config --global rerere.enabled true
git config --global core.autocrlf true
git config --global core.editor "code --wait"
git config --global merge.conflictstyle zdiff3

# Diff/Merge tool
git config --global diff.tool vscode
git config --global difftool.vscode.cmd 'code --wait --diff $LOCAL $REMOTE'
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait --merge $REMOTE $LOCAL $BASE $MERGED'
git config --global mergetool.keepBackup false

# SSH signing
git config --global gpg.format ssh
git config --global user.signingkey "$env:USERPROFILE\.ssh\id_ed25519.pub"
git config --global commit.gpgsign true
git config --global tag.gpgsign true
git config --global gpg.ssh.allowedSignersFile "$env:USERPROFILE/.ssh/allowed_signers"
git config --global core.sshCommand "C:/Windows/System32/OpenSSH/ssh.exe"

# Credential helper
git config --global credential.helper manager

# Gitignore globale
git config --global core.excludesfile "$env:USERPROFILE\.gitignore_global"

# Alias
git config --global alias.st "status -sb"
git config --global alias.co "checkout"
git config --global alias.cm "commit -m"
git config --global alias.amend "commit --amend --no-edit"
git config --global alias.last "log -1 HEAD --stat"
git config --global alias.unstage "reset HEAD --"
git config --global alias.lg "log --graph --abbrev-commit --decorate --format=format:'%C(bold blue)%h%C(reset) - %C(bold green)(%ar)%C(reset) %C(white)%s%C(reset) %C(dim white)- %an%C(reset)%C(bold yellow)%d%C(reset)' --all"
git config --global alias.brs "!f() { git branch --sort=-committerdate --color=always | grep -E '^(\*| ) (main|remotes/origin/main)$'; git branch --sort=-committerdate --color=always | grep -E '^(\*| ) (develop|remotes/origin/develop)$'; git branch --sort=-committerdate --color=always | grep -vE ' (main|develop)$'; }; f"
```

---

*Guida aggiornata a maggio 2026. Le funzionalità di Git e GitHub evolvono: rivisita le configurazioni una volta all'anno per restare allineato con le best practice correnti.*
