# Workflow Professionale per Sviluppatore Singolo
 
Linea guida operativa completa per progetti di sviluppo software gestiti in autonomia, con standard di qualità professionali. Basata sul plugin [everything-claude-code](https://github.com/affaan-m/everything-claude-code).
 
Il documento copre l'intero ciclo di vita del progetto, dallo scoping iniziale al deploy in produzione, con istruzioni esplicite per ogni fase, gestione degli errori, e procedure di emergenza. Nessun passaggio è implicito.
 
---
 
## Principi Generali (da rileggere ogni inizio progetto)
 
1. **Niente codice prima di Fase 0.** Non si apre l'editor finché scope, criteri di "fatto" e ADR iniziali non sono scritti.
2. **`/compact` tra le fasi maggiori.** Tiene il contesto pulito e migliora la qualità dell'output.
3. **`/checkpoint` ogni volta che termini una sotto-attività che funziona.** Il rollback gratis vale ore di debug.
4. **TDD adattivo, non dogmatico.** Soglie di coverage variabili per criticità (vedi Fase 4).
5. **Security review obbligatoria** se il progetto ha auth, input utente, database, o endpoint pubblici.
6. **Documenta mentre lavori, non dopo.** Il "te" fra sei mesi è uno sconosciuto.
7. **Backup mensile degli instinct** fuori dal plugin (vedi Disaster Recovery).
8. **Timebox tutto quello che rischia di diventare un rabbit-hole.** Refactor, debug ostinato, scelte di tooling.
---
 
## FASE 0 — Scoping & Setup
 
Obiettivo: avere chiaro cosa stai costruendo, perché, e con quali vincoli, prima di toccare il codice.
 
### 0.1 — Verifica ambiente
 
Prima di qualsiasi cosa, controlla che il plugin sia operativo:
 
```
/harness-audit
```
 
Se il comando segnala problemi, risolvili prima di procedere. Se il plugin non risponde, controlla `~/.claude/plugins/` e reinstalla seguendo il README di [everything-claude-code](https://github.com/affaan-m/everything-claude-code).
 
Se hai instinct importati da progetti precedenti che vuoi riusare:
 
```
/instinct-import
```
 
### 0.2 — Scrivi il documento di scope
 
Crea `docs/scope.md` nella root del progetto. Deve contenere, in modo esplicito:
 
- **Cosa fa il progetto** (1-2 paragrafi).
- **Cosa NON fa** (lista di non-scope, almeno 3 voci). Questo previene scope creep.
- **Utenti target** (anche se sei tu stesso, descrivi il caso d'uso).
- **Vincoli**: tempo a disposizione, budget cloud/infra, requisiti di compliance (GDPR se ci sono dati personali, accessibilità se è UI pubblica), requisiti di performance (es. "deve girare su 512MB RAM").
- **Definition of Done**: lista puntuale di criteri oggettivi che dichiarano il progetto "completato". Esempio: "deploy su staging funzionante, copertura test >= soglie definite, documentazione API generata, security scan PASS".
Non procedere a 0.3 finché questo file non esiste e non l'hai riletto.
 
### 0.3 — Stima realistica
 
Stima l'effort in ore o giorni per ogni feature principale, poi moltiplica per 1.5. Scrivi le stime in `docs/scope.md` sotto una sezione "Stima". Se il totale supera il tempo che hai realisticamente disponibile, taglia feature, non comprimere stime.
 
### 0.4 — ADR iniziali
 
Crea la cartella `docs/adr/`. Per ogni decisione architetturale non banale (linguaggio, framework, database, modello di deploy), scrivi un file numerato del tipo `0001-scelta-database.md` con la struttura:
 
```
# ADR 0001: <titolo>
Stato: Proposto / Accettato / Sostituito da ADR-XXXX
Data: YYYY-MM-DD
 
## Contesto
Cosa stai decidendo e perché ora.
 
## Opzioni considerate
- A: pro/contro
- B: pro/contro
- C: pro/contro
 
## Decisione
Quale opzione e perché.
 
## Conseguenze
Cosa cambia, quali rischi accetti, cosa rimandi.
```
 
Gli ADR sono il tuo "perché" futuro. In FASE 2 (`/council`) li userai come input.
 
### 0.5 — Inizializza repository
 
```bash
git init
git add .
git commit -m "chore: initial scope and ADRs"
```
 
Crea subito `.gitignore` adeguato allo stack (anche solo abbozzato), e collega un remote privato. **Non procedere senza un remote**: il backup è parte di Fase 0, non un'aggiunta successiva.
 
### Cosa fare se qualcosa fallisce in Fase 0
 
- **Non riesci a definire il non-scope**: il progetto è troppo vago. Riformula come domanda di ricerca, non come progetto, e fai uno spike di 2 ore (vedi sezione "Spike e prototipi").
- **Le stime ti sembrano impossibili**: il progetto è troppo grande per il tempo. Tagliarlo è obbligatorio, non opzionale.
---
 
## FASE 1 — Search-First
 
Obiettivo: non costruire da zero quello che esiste già e funziona meglio.
 
### 1.1 — Comando
 
```
Devo costruire [descrizione del progetto].
Stack: [tecnologie scelte o da scegliere].
Features: [lista delle funzionalità principali].
Usa search-first per trovare librerie, boilerplate o approcci esistenti
prima di scrivere qualsiasi cosa.
```
 
Claude cercherà su npm, GitHub, documentazione ufficiale e presenterà un confronto con raccomandazione.
 
### 1.2 — Verifica licenze
 
Per ogni dipendenza candidata che non sia ovvia (MIT, Apache 2.0, BSD), chiedi esplicitamente:
 
```
Verifica la licenza di [nome libreria] e dimmi se è compatibile
con un progetto [commerciale chiuso / open source MIT / uso interno].
Segnala anche eventuali vincoli di attribution.
```
 
Se trovi una GPL/AGPL e il tuo progetto è chiuso o commerciale, scartala. Documenta la scelta in un ADR (`docs/adr/00XX-licenza-X.md`).
 
### 1.3 — Verifica supply chain
 
Per ogni libreria che entrerà in produzione, controlla:
 
- Ultima release: se è ferma da più di 12 mesi, è un rischio.
- Numero di maintainer attivi: se è una persona sola, è un rischio.
- Issue aperte vs chiuse: se le aperte crescono molto più velocemente delle chiuse, è un rischio.
- CVE note: chiedi a Claude di controllare con `npm audit` (Node), `pip-audit` (Python), o equivalenti.
### Cosa fare se qualcosa fallisce in Fase 1
 
- **Non trovi nulla di adatto**: probabilmente lo scope è troppo specifico. Riformula la ricerca a un livello più astratto ("framework per X" invece di "libreria che fa esattamente Y").
- **Trovi tre cose equivalenti**: vai in Fase 2 con `/council`.
- **Trovi qualcosa che fa il 90% del progetto**: rivaluta lo scope. Forse non serve costruire nulla, basta integrare.
---
 
## FASE 2 — Decisione Architetturale (solo se necessaria)
 
Obiettivo: prendere decisioni architetturali non ovvie con un contraddittorio strutturato.
 
### 2.1 — Quando usarla
 
Usa `/council` quando:
 
- Hai 2+ opzioni in bilico dopo Fase 1.
- La decisione è difficile da invertire (cambio DB, cambio linguaggio, cambio modello di deploy).
- Stai per fare una scelta che ti farà perdere giorni se sbagli.
**Non** usarla per scelte facilmente reversibili (es. quale logger usare, quale CSS framework). Costa contesto e tempo.
 
### 2.2 — Comando
 
```
Ho due opzioni: [Opzione A] vs [Opzione B].
Contesto: [progetto, vincoli, priorità, riferimento ADR pertinenti].
Usa /council.
```
 
Claude convoca 4 voci (Architect, Skeptic, Pragmatist, Critic) in parallelo e fornisce un verdetto con motivazione e dissensi.
 
### 2.3 — Cosa fare con il verdetto
 
1. Leggi anche le voci di minoranza, non solo il verdetto. Spesso il dissenso contiene il rischio reale.
2. Scrivi un ADR (`docs/adr/00XX-...`) con la decisione, i pro/contro emersi, e le condizioni che ti farebbero rivalutare ("se il numero di utenti supera X, riconsideriamo").
3. Se il verdetto non ti convince:
   - Riformula con contesto più stretto e ripeti `/council`.
   - Oppure fai uno spike di max 2 ore per testare l'opzione su cui hai dubbi.
   - Non procedere "tanto poi vediamo": se hai dubbi adesso, ne avrai di peggiori dopo.
### Cosa fare se qualcosa fallisce in Fase 2
 
- **`/council` produce verdetti contraddittori in più run**: la decisione dipende da informazioni che non hai ancora. Spike di 2 ore per raccoglierle.
- **Tutte le opzioni sembrano cattive**: probabilmente il problema è mal posto. Torna a Fase 0 e rivedi lo scope.
---
 
## FASE 3 — Pianificazione
 
Obiettivo: avere una task list ordinata e dipendenze chiare prima di scrivere codice.
 
### 3.1 — Piano completo
 
```
Pianifica l'implementazione completa di [nome progetto].
Stack scelto: [tecnologie].
Features:
- [feature 1]
- [feature 2]
- [feature 3]
Produci: architettura, struttura file, task list ordinata, dipendenze necessarie,
rischi noti per ogni task.
```
 
### 3.2 — Per progetti con più fronti (es. frontend + backend + worker)
 
```
/multi-plan
```
 
### 3.3 — Salva il piano
 
Salva il piano in `docs/plan.md`. Sarà il tuo riferimento operativo. Aggiornalo quando le cose cambiano (e cambieranno).
 
### 3.4 — Compatta il contesto prima di iniziare a sviluppare
 
```
/compact Focus on implementing [nome progetto]: [stack in breve]
```
 
`/compact` è un comando nativo di Claude Code, non del plugin. Pulisce il contesto mantenendo solo quello che serve per la fase successiva.
 
### Cosa fare se qualcosa fallisce in Fase 3
 
- **Il piano è troppo lungo (> 30 task)**: dividi il progetto in milestone, pianifica solo la prima milestone in dettaglio.
- **Le dipendenze formano cicli**: c'è un problema di design. Risolvi prima di iniziare.
---
 
## FASE 4 — Sviluppo (una feature alla volta, TDD adattivo)
 
Obiettivo: implementare le feature una alla volta, con test commisurati alla criticità.
 
### 4.1 — Coverage adattiva (regola sostitutiva dell'80% rigido)
 
| Tipo di codice | Coverage minima | Note |
|---|---|---|
| Auth, pagamenti, sicurezza, dati sensibili | 100% sui path critici | Test anche dei casi di errore |
| Logica di business core | 80% | Unit + integration |
| API/controller | 70% | Integration test sufficienti |
| Glue code, adapter, wiring | 50% o smoke test | Non over-testare |
| Spike e prototipi etichettati come tali | 0% (skip TDD) | Da rifare se promossi a produzione |
 
Scrivi nel piano (Fase 3) quale categoria si applica a ogni feature.
 
### 4.2 — Comando per ogni feature
 
```
Implementa [nome feature].
Segui il tdd-workflow:
- Prima scrivi i test (unit + integration)
- Poi implementa
- Coverage target: [X%] (vedi tabella)
Stack: [tecnologie rilevanti per questa feature].
Vincoli specifici: [es. autenticazione, isolamento dati, rate limiting].
```
 
### 4.3 — Disciplina anti rabbit-hole
 
Imposta un timebox mentale per ogni feature: se sfori del 50%, ti fermi e fai:
 
```
/checkpoint
```
 
Poi rivaluti: la stima era sbagliata, o stai over-engineering? Se è il secondo caso, rollback al checkpoint e taglia.
 
### 4.4 — Checkpoint frequenti
 
Salva un checkpoint ogni volta che hai un test verde nuovo o una sotto-feature funzionante:
 
```
/checkpoint
```
 
### 4.5 — Verifica copertura in tempo reale
 
```
/test-coverage
```
 
Esegui questo comando alla fine di ogni feature, prima di passare alla successiva.
 
### 4.6 — Feature successive
 
```
Adesso implementa [feature successiva].
Segui sempre tdd-workflow secondo la coverage target già definita.
[Eventuali vincoli specifici].
```
 
### 4.7 — Quando saltare il TDD (eccezione esplicita)
 
Salta il TDD solo per:
 
- Spike etichettati come tali in `docs/plan.md`.
- Glue code la cui correttezza è verificabile a colpo d'occhio.
- Script throwaway che non andranno in produzione.
In tutti gli altri casi, TDD non è negoziabile.
 
### Cosa fare se qualcosa fallisce in Fase 4
 
- **I test passano ma il codice non funziona**: i test testano la cosa sbagliata. Riscrivi prima i test.
- **Non riesci a far passare un test per più di 30 minuti**: `/checkpoint` di rollback al verde precedente, poi spike di 1 ora per capire il problema isolatamente.
- **La feature è più grande del previsto**: dividila in sotto-feature e ripianifica (mini Fase 3).
---
 
## FASE 5 — Code Review
 
Obiettivo: ricevere il feedback critico che lavorando da solo non ricevi da nessun collega.
 
### 5.1 — Review generica
 
Dopo ogni blocco di codice scritto e con i test verdi:
 
```
/code-review
```
 
### 5.2 — Review per linguaggio specifico
 
Usa il comando corrispondente al tuo stack quando disponibile:
 
```
/python-review
/go-review
/cpp-review
/rust-review
/kotlin-review
/flutter-review
```
 
### 5.3 — Refactor mirato
 
Se la review evidenzia problemi strutturali:
 
```
/refactor-clean
```
 
**Timebox: 1 ora massimo.** Se sfori, fermati, `/checkpoint`, e considera se il refactor è davvero necessario adesso o può aspettare. Documenta il debito tecnico in `docs/tech-debt.md`.
 
### Cosa fare se qualcosa fallisce in Fase 5
 
- **La review trova problemi gravi (architettura sbagliata)**: rollback al checkpoint pre-feature e ripensa il design. Aggiungi un ADR per spiegare la nuova scelta.
- **La review è "tutto ok" ma tu hai dubbi**: chiedi esplicitamente "trova i tre punti più deboli di questo codice". Le review generiche tendono a essere indulgenti.
---
 
## FASE 6 — Security Review
 
Obiettivo: trovare i problemi di sicurezza prima che lo facciano altri.
 
### 6.1 — Quando è obbligatoria
 
Se il progetto ha **anche solo una** di queste caratteristiche, Fase 6 è obbligatoria:
 
- Autenticazione utente
- Input da utenti
- Query a database
- Endpoint pubblici (HTTP, WebSocket, gRPC)
- Lettura/scrittura file su disco con percorsi parametrizzati
- Esecuzione di comandi shell o codice dinamico
- Gestione di token, credenziali, secret
Per progetti puramente locali, single-user, senza rete, senza input esterni, puoi saltare. Ma documentalo in `docs/scope.md`.
 
### 6.2 — Review con comando dedicato (default)
 
```
/security-review
```
 
Il comando del plugin esegue una review strutturata coprendo gli ambiti standard (secrets, injection, token storage, rate limiting, log leakage, gestione errori, dipendenze).
 
### 6.3 — Aggiungi contesto se serve (follow-up opzionale)
 
Se il progetto ha aspetti che il comando non può inferire da solo, manda un follow-up dopo `/security-review`:
 
```
Aggiungi a quanto già controllato anche: [contesto specifico, es. "ho appena
migrato l'auth da JWT a session cookie, verifica la migrazione",
oppure "abbiamo un endpoint pubblico che accetta upload file, controlla path traversal e MIME spoofing"].
```
 
Usa la prosa solo per il contesto che `/security-review` non può scoprire da solo. Per il resto, fidati del comando.
 
### 6.4 — Scansione automatica
 
```bash
npx ecc-agentshield scan
```
 
AgentShield esegue 1282 test automatici. Risolvi tutti gli HIGH e CRITICAL prima di procedere. I MEDIUM/LOW vanno valutati caso per caso e tracciati in `docs/tech-debt.md` se rinviati.
 
### 6.5 — Cosa fare se trovi un problema critico
 
1. **Fermati immediatamente.** Non procedere a Fase 7.
2. `/checkpoint` per salvare lo stato attuale.
3. Fix focalizzato sul problema, con test che dimostri la correzione.
4. Ri-esegui Fase 6 da zero.
5. Aggiorna `docs/adr/` se la fix ha cambiato decisioni architetturali.
### Cosa fare se qualcosa fallisce in Fase 6
 
- **AgentShield segnala falsi positivi**: documentali in `docs/security-exceptions.md` con motivazione esplicita. Non ignorarli silenziosamente.
- **Trovi problemi sistemici (non singoli bug)**: il design è sbagliato. Rollback al checkpoint pre-feature e ripensa.
---
 
## FASE 7 — Verifica Finale
 
Obiettivo: confermare che il progetto è pronto per il merge prima del PR.
 
### 7.1 — Verification loop completo (default)
 
```
/verification-loop
```
 
Il comando esegue in sequenza: build, typecheck, lint, test con coverage, security scan. Si ferma al primo fallimento e restituisce un report finale PASS/FAIL.
 
### 7.2 — Quality gate rapido
 
Se hai fretta o stai facendo una verifica intermedia (es. fra una feature e l'altra, senza voler ancora il loop completo):
 
```
/quality-gate
```
 
Differenza pratica: `/quality-gate` è il "siamo a posto in questo momento?" da usare durante lo sviluppo, `/verification-loop` è il "siamo pronti per il PR?" da usare a fine milestone.
 
### 7.3 — Cosa fare se il verification-loop fallisce
 
- **Build fail**: `/build-fix` poi ri-esegui il loop.
- **Lint fail**: applica i fix automatici (`--fix`), poi ri-esegui. Se restano errori manuali, risolvili uno alla volta.
- **Test fail**: torna a Fase 4 sulla feature che fa fallire i test.
- **Coverage sotto soglia**: aggiungi i test mancanti, non abbassare la soglia.
- **Security fail**: torna a Fase 6.
### Cosa fare se non riesci a passare il verification-loop dopo 3 tentativi
 
Stop. C'è un problema strutturale. Apri `docs/tech-debt.md` o un nuovo ADR per documentare cosa stai vedendo, poi spike di 2 ore per capire la causa radice prima di altri tentativi.
 
---
 
## FASE 8 — Git & Pull Request
 
Obiettivo: registrare il lavoro nel repository in modo pulito e tracciabile.
 
### 8.1 — Commit
 
```
/prp-commit
```
 
Crea commit seguendo conventional commits (`feat:`, `fix:`, `chore:`, `docs:`, `refactor:`, `test:`).
 
### 8.2 — Pull request
 
```
/prp-pr
```
 
Genera il PR con descrizione completa, test plan e checklist. Anche da solo, apri PR invece di merge diretto su main: avere lo storico delle PR è utile per ricostruire il "perché" di una modifica fra mesi.
 
### 8.3 — Self-review del PR
 
Anche se il PR sei tu, fallo:
 
```
/review-pr
```
 
Leggi i commenti generati, applica i fix sensati, ignora con motivazione i nitpick.
 
### 8.4 — Merge
 
Merge solo dopo:
 
- Verification loop PASS (Fase 7).
- Self-review completata (8.3).
- CI verde (se hai pipeline CI).
### Cosa fare se qualcosa fallisce in Fase 8
 
- **Conflict di merge**: risolvi sul branch, ri-esegui Fase 7 dopo la risoluzione.
- **CI fail su qualcosa che in locale passa**: differenza ambiente. Investiga prima di mergere.
---
 
## FASE 8.5 — Deploy, Observability & Rollback
 
Obiettivo: andare in produzione in modo controllato e poter tornare indietro.
 
### 8.5.1 — Ambienti separati
 
Configurazione minima:
 
- **Dev (locale)**: tu mentre lavori.
- **Staging**: ambiente identico a produzione, dati finti o anonimizzati.
- **Produzione**: l'ambiente reale.
Anche per progetti piccoli, almeno staging + produzione separate. Configurazioni distinte via variabili d'ambiente, mai hardcoded.
 
### 8.5.2 — Pre-deploy checklist (esegui ogni volta)
 
1. Backup del database di produzione (se esiste).
2. Tag di versione su Git (`git tag v0.X.Y`).
3. Verification loop su staging deployment.
4. Smoke test manuale dei flussi critici su staging.
5. Annota la versione precedente: ti servirà per il rollback.
### 8.5.3 — Deploy
 
Procedura specifica del tuo stack/infra. Documentala in `docs/deploy.md` con istruzioni copia-incolla, non a memoria.
 
### 8.5.4 — Observability minima
 
Anche per progetti piccoli, almeno:
 
- **Log strutturati** (JSON) con timestamp, livello, request ID se HTTP.
- **Log centralizzati** se il deploy è multi-istanza (anche solo un file ruotato in un volume).
- **Health check endpoint** (`/health` o equivalente).
- **Almeno una metrica** che ti dice se l'app è viva (uptime, error rate).
Configura un alert (anche solo email) se l'health check fallisce per più di N minuti.
 
### 8.5.5 — Procedura di rollback (scrivila, non improvvisarla)
 
Crea `docs/rollback.md` con:
 
- Comando esatto per redeployare la versione precedente.
- Procedura di rollback del database (se ci sono migration non backward-compatible, hai un problema; documenta come gestirlo).
- Tempo stimato di rollback.
- Chi avvisare (anche solo "scrivi un post mortem in `docs/incidents/`").
### 8.5.6 — Backup automatici
 
Database e file utente: backup automatico almeno giornaliero, copiato fuori dalla macchina di produzione (S3, altro provider, NAS personale). Verifica mensile che il restore funzioni davvero (un backup non testato non è un backup).
 
### Cosa fare se qualcosa fallisce in Fase 8.5
 
- **Deploy fallisce a metà**: esegui la procedura di rollback (8.5.5) prima di indagare.
- **Health check rosso post-deploy**: rollback immediato. Investiga in staging.
- **Backup mai testato fallisce al primo restore reale**: incidente serio. Scrivi post mortem, fissa la procedura, testa di nuovo.
---
 
## FASE 9 — Documentazione
 
Obiettivo: rendere il progetto leggibile dal "te" futuro.
 
### 9.1 — Update docs
 
```
/update-docs
```
 
### 9.2 — Aggiorna a mano se necessario
 
`/update-docs` è bravo ma non onnisciente. Verifica e completa:
 
- **README.md**: come avviare il progetto in locale, comandi principali, link a `docs/`.
- **CHANGELOG.md**: cosa è cambiato in questa release. Segui [Keep a Changelog](https://keepachangelog.com/).
- **ADR nuovi**: ogni decisione architetturale presa durante lo sviluppo.
- **API docs** (se esiste un'API): generate da codice (OpenAPI, JSDoc, ecc.) o scritte a mano.
### 9.3 — Commit della documentazione
 
Documentazione sempre nello stesso PR del codice che descrive. Se documenti dopo, in un PR separato, prima o poi salti il passo.
 
---
 
## FASE 10 — Fine Sessione
 
Obiettivo: lasciare il sistema in uno stato che ti permette di riprendere senza perdere contesto.
 
### 10.1 — Comandi di chiusura (in quest'ordine)
 
```
/learn
/save-session
/evolve
```
 
- `/learn` estrae pattern dalla sessione corrente.
- `/save-session` salva il contesto per la sessione successiva.
- `/evolve` fa evolvere gli instinct esistenti in base a quanto appreso.
### 10.2 — Backup degli instinct (mensile)
 
Una volta al mese, esporta gli instinct e copiali fuori dal plugin:
 
```
/instinct-export
```
 
Poi sposta il file generato in un repository separato (es. `~/dev/instinct-backups/`) versionato con Git. Se il plugin un giorno cambia formato o si rompe, hai una copia.
 
### 10.3 — Riprendere una sessione
 
A inizio sessione successiva:
 
```
/resume-session
```
 
---
 
## Spike e Prototipi (regola di esclusione)
 
Uno **spike** è un'esplorazione di max 2 ore per rispondere a una domanda tecnica specifica.
 
Per uno spike:
 
1. Scrivi in `docs/spikes/YYYY-MM-DD-domanda.md` la domanda a cui vuoi rispondere e il timebox.
2. Salta TDD, salta code review, salta security review.
3. Al termine, scrivi la risposta nel file.
4. **Cancella o riscrivi il codice dello spike.** Non promuoverlo a produzione senza passare dal workflow completo.
Se uno spike va oltre il timebox, fermati, scrivi cosa hai capito, e decidi se serve un altro spike o se è il momento di pianificare seriamente.
 
---
 
## Disaster Recovery e Backup
 
Single point of failure: tu. Mitigazioni:
 
- **Push su Git almeno a fine giornata.** Idealmente dopo ogni feature completata.
- **Remote privato su provider diverso da quello dove vivi i tuoi token di auth principali.** Ridondanza.
- **Backup mensile di**: database, file utente, instinct (`/instinct-export`), file di sessione del plugin.
- **Test di restore trimestrale**: prendi un backup, ripristinalo in un ambiente isolato, verifica che funzioni.
- **Password/token in un password manager**, non in file sul disco. Token di accesso al cloud su un secondo dispositivo o su carta in cassaforte.
---
 
## Comandi Utili in Qualsiasi Momento
 
| Comando | Quando usarlo |
|---|---|
| `/instinct-status` | Vedere gli instinct appresi finora |
| `/instinct-import` | Importare instinct da un altro progetto |
| `/instinct-export` | Esportare gli instinct correnti (fai backup mensile) |
| `/resume-session` | Riprendere una sessione interrotta |
| `/checkpoint` | Salvare un punto di ripristino durante lo sviluppo |
| `/build-fix` | Fix automatico di errori di build |
| `/update-docs` | Aggiornare la documentazione dopo modifiche al codice |
| `/loop-start` | Avviare un loop iterativo su un task |
| `/model-route` | Scegliere il modello ottimale per il task corrente |
| `/harness-audit` | Audit completo della configurazione del plugin |
| `/skill-create` | Creare una nuova skill personalizzata |
| `/multi-execute` | Eseguire task in parallelo su più fronti |
| `/multi-plan` | Pianificare progetti multi-fronte |
| `/verification-loop` | Verification completo pre-PR (build + typecheck + lint + test + security) |
| `/security-review` | Security review strutturata (default per Fase 6) |
| `/quality-gate` | Quality gate rapido (verifica intermedia durante lo sviluppo) |
| `/test-coverage` | Verificare la copertura test |
| `/code-review` | Code review generica |
| `/refactor-clean` | Refactor mirato (timebox 1h) |
| `/council` | Decisione architetturale strutturata |
| `/prp-commit` | Commit conventional |
| `/prp-pr` | Generazione PR completa |
| `/review-pr` | Review di un PR esistente |
| `/learn` | Estrai pattern dalla sessione |
| `/save-session` | Salva contesto sessione |
| `/evolve` | Fa evolvere gli instinct |
| `/compact` | Comando nativo Claude Code, pulisce il contesto |
 
---
 
## Tabella Riassuntiva: Cosa Fare in Ogni Fase
 
| Fase | Output atteso | Comando chiave | Salta se |
|---|---|---|---|
| 0 — Scoping | `docs/scope.md`, ADR iniziali, repo + remote | `/harness-audit` | MAI |
| 1 — Search-first | Confronto librerie, scelte documentate | `search-first` | MAI |
| 2 — Council | ADR di decisione | `/council` | Decisione facilmente reversibile |
| 3 — Pianificazione | `docs/plan.md` | `/multi-plan`, `/compact` | MAI |
| 4 — Sviluppo | Codice con test, checkpoint frequenti | `/checkpoint`, `/test-coverage` | MAI (ma TDD adattivo) |
| 5 — Code review | Review applicata, debito tracciato | `/code-review`, `/refactor-clean` | MAI |
| 6 — Security | Scan PASS, eccezioni documentate | `/security-review` + `npx ecc-agentshield scan` | Progetto puramente locale single-user senza rete |
| 7 — Verifica | Verification loop PASS | `/verification-loop` | MAI |
| 8 — Git & PR | PR aperta e self-reviewata | `/prp-commit`, `/prp-pr`, `/review-pr` | MAI |
| 8.5 — Deploy | Deploy + observability + rollback documentato | (specifico stack) | Progetto non va in produzione |
| 9 — Docs | README, CHANGELOG, ADR aggiornati | `/update-docs` | MAI |
| 10 — Chiusura | Sessione salvata, instinct evoluti | `/learn`, `/save-session`, `/evolve` | MAI |
 
---
 
## Anti-Pattern da Evitare
 
1. **Saltare Fase 0 perché "è un progetto piccolo".** I progetti piccoli sono quelli dove perdi più tempo per mancanza di scope.
2. **Coverage 80% ovunque.** Sui path critici è poco, sui glue è troppo.
3. **TDD anche per spike.** Vanifica lo spike.
4. **Refactor "veloce" senza timebox.** Diventa sempre lento.
5. **Saltare security review perché "è solo per me".** "Solo per me" oggi è "esposto a internet" fra tre mesi.
6. **Documentare dopo.** Non lo farai.
7. **Backup mai testati.** Non sono backup.
8. **Merge diretto su main senza PR.** Perdi lo storico delle decisioni.
9. **Instinct mai esportati.** Quando il plugin si rompe, hai perso mesi di lavoro.
10. **Fidarsi di `/code-review` come unica review.** È indulgente. Chiedi sempre i tre punti più deboli.
---
 
## Quick Reference: Sequenza Standard di una Feature
 
Per una feature media in un progetto già pianificato (Fase 0-3 già fatte):
 
```
1. /compact "Focus on implementing <feature>"
2. Implementa <feature> seguendo tdd-workflow, coverage target X%
3. /checkpoint
4. /test-coverage
5. /code-review (o variante per linguaggio)
6. /refactor-clean (se serve, timebox 1h)
7. (se feature security-relevant) /security-review + npx ecc-agentshield scan
8. /quality-gate
9. /update-docs
10. /prp-commit
11. /checkpoint
```
 
A fine giornata o fine sessione di lavoro:
 
```
12. /learn
13. /save-session
14. /evolve
15. git push
```
 
A fine milestone (gruppo di feature):
 
```
16. /prp-pr
17. /review-pr
18. /verification-loop
19. Merge
20. Deploy seguendo Fase 8.5
```
 
---
 
## Manutenzione di Questo Documento
 
Questo workflow non è scolpito nella pietra. Quando una fase ti rallenta in modo sistematico, **non saltarla silenziosamente**: aggiorna il documento spiegando cosa hai cambiato e perché. Idealmente, aggiungi una entry nel CHANGELOG personale del workflow stesso.
 
Revisione consigliata: ogni 3 mesi o ogni cambio significativo di stack.
 
