---
title: "Come Creare un Sito Vetrina Professionale nel 2026: Guida Completa con Astro, GitHub e Netlify"
description: "Guida passo-passo per creare un sito vetrina professionale, veloce e SEO-friendly nel 2026. Stack Astro 5 + Tailwind 4, deploy automatico su Netlify, dominio personalizzato, accessibilità, sicurezza e manutenzione continua. Adatta ai principianti, aggiornata e completa."
slug: "creare-sito-vetrina-professionale-astro-netlify-2026"
date: 2026-05-12
updated: 2026-05-12
author: "Matteo Rizzuto"
reading_time: "28 min"
language: "it-IT"
category: "Guide"
keywords:
  - creare sito vetrina
  - sito vetrina professionale
  - guida Astro 2026
  - deploy Netlify GitHub
  - sito web aziendale fai da te
  - hosting gratis Netlify
  - sito statico veloce SEO
  - Tailwind CSS guida italiano
  - Node.js sito web
canonical: "https://www.tuosito.com/guide/creare-sito-vetrina-professionale-astro-netlify-2026"
og_image: "/og/sito-vetrina-astro-netlify-2026.jpg"
og_type: "article"
schema_type: "HowTo"
robots: "index, follow, max-image-preview:large"
---

# Come Creare un Sito Vetrina Professionale nel 2026: Guida Completa con Astro, GitHub e Netlify

> **TL;DR** — In questa guida costruisci passo-passo un **sito vetrina professionale, veloce e SEO-friendly**, ospitato gratis, con dominio personalizzato e HTTPS. Stack 2026: **Astro 5 + Tailwind CSS 4 + Netlify**. Tempo richiesto: **un pomeriggio** se segui l'ordine. Nessuna esperienza di programmazione richiesta.

*Ultimo aggiornamento: 12 maggio 2026 · Tempo di lettura stimato: ~28 minuti · Livello: principiante → intermedio*

---

## Cosa imparerai in questa guida

Alla fine sarai in grado di:

- Installare e configurare correttamente **Node.js, Git e VS Code**
- Creare un progetto **Astro 5 con Tailwind 4 e TypeScript**
- Versionare il codice con **Git e GitHub** usando un workflow professionale
- Pubblicare il sito su **Netlify** con **deploy continuo** da GitHub
- Collegare un **dominio personalizzato con HTTPS automatico**
- Ottimizzare il sito per **SEO, performance, accessibilità e sicurezza**
- Aggiungere form di contatto, analytics rispettosi della privacy e CI/CD
- Mantenere il sito nel tempo seguendo una routine chiara

**Prerequisiti**: un computer con Windows, macOS o Linux, una connessione internet, un'email valida. Niente altro.

---

## Indice

1. [Panoramica e scelte tecniche](#1-panoramica-e-scelte-tecniche)
2. [Strumenti da installare](#2-strumenti-da-installare)
3. [Configurazione account](#3-configurazione-account)
4. [Creazione del progetto Astro 5](#4-creazione-del-progetto-astro-5)
5. [Struttura del progetto](#5-struttura-del-progetto)
6. [Workflow di sviluppo quotidiano](#6-workflow-di-sviluppo-quotidiano)
7. [Workflow Git professionale](#7-workflow-git-professionale)
8. [Pubblicazione su GitHub](#8-pubblicazione-su-github)
9. [Deploy su Netlify](#9-deploy-su-netlify)
10. [Dominio personalizzato e HTTPS](#10-dominio-personalizzato-e-https)
11. [SEO on-page e dati strutturati](#11-seo-on-page-e-dati-strutturati)
12. [Accessibilità (WCAG)](#12-accessibilità-wcag)
13. [Analytics e monitoraggio](#13-analytics-e-monitoraggio)
14. [Performance e Core Web Vitals](#14-performance-e-core-web-vitals)
15. [Form di contatto](#15-form-di-contatto)
16. [Sicurezza, CSP e privacy](#16-sicurezza-csp-e-privacy)
17. [CMS opzionale per non sviluppatori](#17-cms-opzionale-per-non-sviluppatori)
18. [CI/CD con GitHub Actions](#18-cicd-con-github-actions)
19. [Test automatici end-to-end](#19-test-automatici-end-to-end)
20. [`llms.txt`: ottimizzazione per motori AI](#20-llmstxt-ottimizzazione-per-motori-ai)
21. [Checklist pre-lancio](#21-checklist-pre-lancio)
22. [Manutenzione continua](#22-manutenzione-continua)
23. [Risoluzione problemi (FAQ)](#23-risoluzione-problemi-faq)
24. [Risorse utili](#24-risorse-utili)

---

## 1. Panoramica e scelte tecniche

### Cos'è un sito vetrina

Un **sito vetrina** è un sito web di poche pagine pensato per presentare un'attività, un professionista o un prodotto. Non vende online (non è un e-commerce) e non offre funzionalità complesse: il suo scopo è **comunicare con chiarezza** chi sei, cosa fai e come farti contattare.

I requisiti chiave di un buon sito vetrina nel 2026 sono **velocità di caricamento sotto i 2 secondi**, **design responsive**, **ottimizzazione SEO**, **accessibilità conforme alle linee guida WCAG**, e **costo di gestione vicino a zero**.

### Stack tecnologico (e perché queste scelte)

| Strumento | Scelta | Perché |
|---|---|---|
| **Framework** | **Astro 5** | Genera HTML statico, zero JavaScript di default, ideale per siti vetrina e SEO |
| **Linguaggio** | **TypeScript** | Sicurezza in più rispetto a JavaScript puro, standard nei progetti seri |
| **CSS** | **Tailwind CSS 4** | Utility-first, design system integrato, build velocissima con il motore Oxide |
| **Versionamento** | **Git + GitHub** | Standard del settore, cronologia delle modifiche, backup remoto gratuito |
| **Hosting** | **Netlify** | Gratuito per progetti personali, CDN globale, HTTPS automatico, deploy continuo |
| **Editor** | **Visual Studio Code** | Gratuito, leggero, estensioni eccellenti per Astro e Tailwind |
| **Form** | **Netlify Forms** | 100 invii/mese gratis, zero backend da gestire |
| **Analytics** | **Plausible** o **Umami** | Rispettano la privacy, niente cookie banner |
| **CI/CD** | **GitHub Actions** | Gratuito sui repository pubblici, lint e build automatici |
| **Test E2E** | **Playwright** | Test affidabili in tutti i browser principali |

### Perché Astro e non Next.js, WordPress o React puro?

**Astro** è la scelta migliore in assoluto per un sito vetrina nel 2026:

- **Astro** genera HTML statico con zero JavaScript di default. Risultato: Lighthouse 100/100 senza sforzo.
- **Next.js** è ottimo per applicazioni web interattive ma pesante e sovradimensionato per un vetrina.
- **WordPress** richiede manutenzione, backup, aggiornamenti continui, hosting a pagamento, plugin di sicurezza.
- **React puro** non è un framework: dovresti configurare routing, SSG, SEO manualmente.

Risultato per il tuo sito: **caricamento sotto il secondo**, **costo zero**, **manutenzione minima**.

---

## 2. Strumenti da installare

Installa **rigorosamente in questo ordine**, perché ogni strumento dipende dal precedente.

### 2.1 Node.js LTS

Node.js è l'ambiente di esecuzione JavaScript necessario per usare Astro.

1. Vai su [nodejs.org](https://nodejs.org)
2. Scarica la versione **LTS** (Long Term Support, attualmente Node 22)
3. Installa con tutte le opzioni di default
4. Verifica aprendo il **Prompt dei comandi** (Windows) o il **Terminale** (macOS/Linux):

   ```bash
   node --version
   npm --version
   ```

   Devi vedere due numeri di versione, ad esempio `v22.x.x` e `10.x.x`.

### 2.2 Git

Git è il sistema di controllo versione che ti permette di tracciare ogni modifica.

1. Scarica da [git-scm.com/downloads](https://git-scm.com/downloads)
2. Installa con le opzioni di default (lascia "Git from the command line and also from 3rd-party software")
3. Verifica:

   ```bash
   git --version
   ```

### 2.3 Visual Studio Code

L'editor di codice gratuito di Microsoft, lo standard de facto per lo sviluppo web.

1. Scarica da [code.visualstudio.com](https://code.visualstudio.com)
2. Installa e apri

### 2.4 Estensioni VS Code essenziali

Apri VS Code → icona **Estensioni** (`Ctrl+Shift+X` su Windows/Linux, `Cmd+Shift+X` su macOS) → installa una per una:

- **Astro** (di astro-build) — sintassi e snippet per i file `.astro`
- **Tailwind CSS IntelliSense** — autocompletamento delle classi Tailwind
- **ESLint** — segnala errori nel codice JavaScript/TypeScript
- **Prettier** — formattazione automatica del codice
- **GitLens** — visualizza chi ha cambiato cosa nel codice
- **Error Lens** — mostra errori direttamente accanto alla riga
- **Auto Rename Tag** — rinomina automaticamente i tag HTML accoppiati
- **Path Intellisense** — autocompletamento dei percorsi dei file
- **axe Accessibility Linter** — segnala problemi di accessibilità mentre scrivi

### 2.5 Browser per il test

- **Chrome** o **Edge** (per Lighthouse e DevTools)
- **Firefox** (per controllare la compatibilità cross-browser)

---

## 3. Configurazione account

### 3.1 Configura Git con la tua identità

Apri il terminale (in VS Code: `Terminale → Nuovo Terminale`) e incolla:

```bash
git config --global user.name "Il Tuo Nome"
git config --global user.email "tua@email.com"
git config --global init.defaultBranch main
git config --global pull.rebase false
```

> Usa la **stessa email** che userai per GitHub: serve a far apparire i tuoi commit nel grafico delle attività.

### 3.2 Crea l'account GitHub

1. Vai su [github.com](https://github.com) → **Sign up**
2. Verifica l'email
3. **Attiva la 2FA** (autenticazione a due fattori): `Settings → Password and authentication → Two-factor authentication`. È **obbligatoria** per molte funzioni dal 2024 ed è una pratica di sicurezza basilare.

### 3.3 Crea l'account Netlify

1. Vai su [netlify.com](https://netlify.com) → **Sign up**
2. Scegli **"Sign up with GitHub"**: collegamento immediato senza creare un'altra password
3. Autorizza Netlify ad accedere a GitHub

### 3.4 Configura SSH per GitHub (consigliato)

Ti permette di pushare senza inserire credenziali ogni volta.

```bash
ssh-keygen -t ed25519 -C "tua@email.com"
```

Premi Invio tre volte per accettare i default. Poi copia la chiave pubblica negli appunti:

- **Windows**: `clip < ~/.ssh/id_ed25519.pub`
- **macOS**: `pbcopy < ~/.ssh/id_ed25519.pub`
- **Linux**: `cat ~/.ssh/id_ed25519.pub` e copia manualmente

Vai su GitHub → `Settings → SSH and GPG keys → New SSH key` → incolla → **Save**.

Testa il collegamento:

```bash
ssh -T git@github.com
```

Dovresti leggere `Hi nome-utente! You've successfully authenticated`.

---

## 4. Creazione del progetto Astro 5

### 4.1 Scegli una cartella di lavoro

Crea una cartella `progetti` nei tuoi Documenti, ad esempio:

- Windows: `C:\Users\TuoNome\Documenti\progetti`
- macOS/Linux: `~/Documenti/progetti`

Apri quella cartella **in VS Code** (`File → Apri Cartella`).

### 4.2 Crea il progetto Astro

Apri il terminale integrato di VS Code (`Ctrl+ò` su Windows/Linux, `` Ctrl+` `` su macOS) ed esegui:

```bash
npm create astro@latest
```

Rispondi alle domande del wizard così:

- **Dove creare il progetto?** → `mio-sito-vetrina`
- **Template iniziale** → scegli quello **minimo / vuoto** (etichetta esatta varia tra versioni)
- **Installare le dipendenze?** → **Sì**
- **Usare TypeScript?** → **Sì**, modalità **strict**
- **Inizializzare un repository Git?** → **Sì**

### 4.3 Entra nel progetto e avvialo

```bash
cd mio-sito-vetrina
npm run dev
```

Apri il browser su **http://localhost:4321** e dovresti vedere la pagina di benvenuto di Astro.

Per fermare il server: `Ctrl+C` nel terminale.

### 4.4 Aggiungi Tailwind CSS 4

Dalla versione 4, Tailwind si integra come **plugin Vite** (non più tramite il pacchetto `@astrojs/tailwind`):

```bash
npm install tailwindcss @tailwindcss/vite
```

Modifica `astro.config.mjs`:

```js
import { defineConfig } from 'astro/config';
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
  site: 'https://www.tuosito.com',
  vite: { plugins: [tailwindcss()] },
});
```

Crea (o modifica) `src/styles/global.css`:

```css
@import "tailwindcss";
```

Importa il CSS nel tuo layout principale (`src/layouts/BaseLayout.astro`):

```astro
---
import '../styles/global.css';
---
```

### 4.5 Aggiungi sitemap e formattatore

```bash
npx astro add sitemap
npm install --save-dev prettier prettier-plugin-astro prettier-plugin-tailwindcss
```

Crea `.prettierrc.json` nella root del progetto:

```json
{
  "plugins": ["prettier-plugin-astro", "prettier-plugin-tailwindcss"],
  "overrides": [{ "files": "*.astro", "options": { "parser": "astro" } }],
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "printWidth": 100
}
```

---

## 5. Struttura del progetto

Questa è la struttura professionale consigliata:

```
mio-sito-vetrina/
├── public/                  # File serviti tal quali (favicon, robots.txt)
│   ├── favicon.svg
│   ├── robots.txt
│   ├── llms.txt
│   └── images/
├── src/
│   ├── assets/              # Immagini ottimizzate da Astro
│   ├── components/          # Componenti riutilizzabili
│   │   ├── Header.astro
│   │   ├── Footer.astro
│   │   ├── Hero.astro
│   │   ├── SEO.astro
│   │   └── ContactForm.astro
│   ├── layouts/
│   │   └── BaseLayout.astro
│   ├── pages/               # Ogni file = un URL
│   │   ├── index.astro      # → /
│   │   ├── chi-siamo.astro  # → /chi-siamo
│   │   ├── servizi.astro    # → /servizi
│   │   ├── contatti.astro   # → /contatti
│   │   └── 404.astro
│   ├── styles/
│   │   └── global.css
│   └── content/             # (Opzionale) contenuti in Markdown
├── tests/                   # Test end-to-end con Playwright
├── .github/workflows/       # CI/CD GitHub Actions
├── .env.example
├── .gitignore
├── .prettierrc.json
├── astro.config.mjs
├── netlify.toml
├── package.json
├── playwright.config.ts
├── tsconfig.json
└── README.md
```

### Pagine consigliate per un sito vetrina SEO-ottimizzato

1. **Home** (`/`) — proposta di valore + CTA forte
2. **Chi siamo / Chi sono** (`/chi-siamo`) — storia, valori, team (segnali E-E-A-T per Google)
3. **Servizi** (`/servizi`) — una pagina per ogni servizio principale (ottimo per SEO long-tail)
4. **Portfolio / Casi studio** (`/portfolio`) — prove sociali, screenshot, testimonianze
5. **Blog** (`/blog`) — opzionale ma ottimo per la SEO organica
6. **Contatti** (`/contatti`) — form + dati strutturati `LocalBusiness`
7. **Privacy policy** (`/privacy`) — obbligatoria per legge se raccogli dati
8. **Cookie policy** (`/cookie`) — obbligatoria se usi cookie non tecnici
9. **404** — pagina di errore personalizzata con link utili

---

## 6. Workflow di sviluppo quotidiano

### La routine professionale

**All'inizio di ogni sessione di lavoro:**

1. Apri VS Code nella cartella del progetto
2. Apri il terminale integrato
3. Scarica eventuali modifiche remote: `git pull`
4. Avvia il dev server: `npm run dev`
5. Tieni aperto il sito nel browser per vedere le modifiche **in tempo reale**

### Comandi che userai sempre

```bash
npm run dev      # Sviluppo locale su http://localhost:4321
npm run build    # Build di produzione nella cartella ./dist
npm run preview  # Anteprima locale del build di produzione
```

### La regola d'oro: piccoli incrementi

Modifica una cosa → guardala nel browser → se va bene, **committa** → passa al prossimo pezzo. Mai accumulare ore di modifiche prima di committare.

---

## 7. Workflow Git professionale

### Concetti base

- **Repository (repo)** — la cartella del progetto sotto controllo versione
- **Commit** — uno snapshot del progetto in un dato momento
- **Branch** — una linea di sviluppo parallela
- **Push** — invia i commit a GitHub
- **Pull** — scarica i commit da GitHub
- **Pull Request (PR)** — richiesta di unire un branch nel principale

### Strategia di branching

**Se lavori da solo** su un sito vetrina, puoi semplicemente committare su `main` con commit piccoli e frequenti. Il workflow con feature branch è cerimonia inutile per un singolo sviluppatore.

**Se siete due o più persone** o lavori per un cliente che vuole approvare le modifiche prima della pubblicazione:

- **`main`** — sempre stabile, riflette il sito in produzione
- **`feature/nome-funzionalità`** — un branch per ogni novità

```bash
git checkout -b feature/sezione-portfolio
# ...lavori, modifichi file...
git add .
git commit -m "feat: aggiunta sezione portfolio in home"
git push origin feature/sezione-portfolio
# Su GitHub: apri Pull Request → Merge → cancella branch
```

### Conventional Commits

Usa **sempre** questo formato per i messaggi:

| Prefisso | Quando | Esempio |
|---|---|---|
| `feat:` | Nuova funzionalità | `feat: aggiunta pagina servizi` |
| `fix:` | Correzione bug | `fix: form contatti non inviava` |
| `style:` | Modifiche estetiche | `style: nuovo colore primario` |
| `refactor:` | Riscrittura senza cambi funzionali | `refactor: estratto componente Hero` |
| `docs:` | Documentazione | `docs: aggiornato README` |
| `chore:` | Manutenzione, dipendenze | `chore: aggiornate dipendenze` |
| `perf:` | Miglioramento prestazioni | `perf: lazy load immagini portfolio` |
| `test:` | Aggiunta o modifica test | `test: e2e form contatti` |
| `ci:` | Configurazione CI | `ci: aggiunto job di build su PR` |

**Buon messaggio**: imperativo presente, sotto i 72 caratteri, descrive **cosa cambia** non **come**.

### Comandi Git essenziali

```bash
git status                    # Cosa è cambiato
git diff                      # Mostra le differenze
git add .                     # Mette tutto in staging
git add nome-file             # Mette in staging solo un file
git commit -m "messaggio"     # Crea il commit
git log --oneline             # Cronologia compatta
git push                      # Invia a GitHub
git pull                      # Scarica da GitHub
git checkout -b nome-branch   # Crea e passa a un nuovo branch
git checkout main             # Torna su main
git merge nome-branch         # Unisce un branch al corrente
git stash                     # Mette via le modifiche temporaneamente
git reset --hard HEAD         # Annulla TUTTE le modifiche non committate (attento!)
```

### File `.gitignore`

Verifica che contenga almeno:

```
node_modules/
dist/
.env
.env.*
!.env.example
.DS_Store
*.log
.netlify/
.vscode/settings.json
playwright-report/
test-results/
```

---

## 8. Pubblicazione su GitHub

### 8.1 Crea il repository su GitHub

1. Vai su [github.com/new](https://github.com/new)
2. **Repository name**: `mio-sito-vetrina`
3. **Description**: una frase chiara (es. *"Sito vetrina aziendale realizzato con Astro e Tailwind"*)
4. **Public** (consigliato per portfolio personale) o **Private** (per clienti)
5. **NON** spuntare README, .gitignore, license — sono già nel progetto
6. Click **Create repository**

### 8.2 Collega il progetto locale a GitHub

Con SSH (consigliato):

```bash
git remote add origin git@github.com:tuo-username/mio-sito-vetrina.git
git branch -M main
git push -u origin main
```

Con HTTPS:

```bash
git remote add origin https://github.com/tuo-username/mio-sito-vetrina.git
git branch -M main
git push -u origin main
```

### 8.3 Verifica

Ricarica la pagina del repository: dovresti vedere tutti i file caricati.

### 8.4 Crea un README utile

Sostituisci il README di default con qualcosa di descrittivo (utile sia per chi visita il repo, sia per la SEO se il repo è pubblico):

```markdown
# Mio Sito Vetrina

Sito vetrina realizzato con Astro 5, Tailwind 4 e deploy automatico su Netlify.

## Stack
- Astro 5 + TypeScript strict
- Tailwind CSS 4
- Netlify (hosting + forms + analytics)
- GitHub Actions (CI)
- Playwright (E2E)

## Sviluppo locale

\`\`\`bash
npm install
npm run dev
\`\`\`

## Build

\`\`\`bash
npm run build
\`\`\`

## Licenza
© 2026 Mio Nome — tutti i diritti riservati.
```

---

## 9. Deploy su Netlify

### 9.1 Importa il progetto

1. Vai su [app.netlify.com](https://app.netlify.com)
2. **Add new site → Import an existing project**
3. **Deploy with GitHub** → autorizza
4. Seleziona `mio-sito-vetrina`

### 9.2 Configurazione del build

Netlify rileva Astro automaticamente. Conferma:

- **Branch to deploy**: `main`
- **Build command**: `npm run build`
- **Publish directory**: `dist`
- **Node version**: `22`

Click **Deploy**. In 1-2 minuti il sito è online su un URL tipo `random-name-12345.netlify.app`.

### 9.3 Personalizza il nome del sito

`Site overview → Site settings → Change site name` → ad esempio `studio-rossi-architetti` diventa `studio-rossi-architetti.netlify.app`.

### 9.4 Crea il file `netlify.toml`

Centralizza la configurazione di build, redirect e security headers. Nella root del progetto:

```toml
[build]
  command = "npm run build"
  publish = "dist"

[build.environment]
  NODE_VERSION = "22"

# Cache aggressiva per gli asset versionati
[[headers]]
  for = "/_astro/*"
  [headers.values]
    Cache-Control = "public, max-age=31536000, immutable"

[[headers]]
  for = "/images/*"
  [headers.values]
    Cache-Control = "public, max-age=31536000, immutable"

# Headers di sicurezza per tutte le pagine
[[headers]]
  for = "/*"
  [headers.values]
    X-Frame-Options = "DENY"
    X-Content-Type-Options = "nosniff"
    Referrer-Policy = "strict-origin-when-cross-origin"
    Permissions-Policy = "camera=(), microphone=(), geolocation=(), interest-cohort=()"
    Strict-Transport-Security = "max-age=63072000; includeSubDomains; preload"
    Content-Security-Policy = "default-src 'self'; script-src 'self' 'unsafe-inline' https://plausible.io; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self' data:; connect-src 'self' https://plausible.io; frame-ancestors 'none'; base-uri 'self'; form-action 'self' https://*.netlify.com"

# Redirect www → root (o viceversa: scegline UNA)
[[redirects]]
  from = "https://www.tuosito.com/*"
  to = "https://tuosito.com/:splat"
  status = 301
  force = true
```

> **Importante CSP**: la riga `Content-Security-Policy` va personalizzata in base ai servizi che usi (analytics, font, embed YouTube, ecc.). Testala con [csp-evaluator.withgoogle.com](https://csp-evaluator.withgoogle.com).

Committa e pusha — Netlify ricostruirà in automatico.

### 9.5 Deploy automatici e preview

Ad ogni `git push` su `main`, Netlify ricostruisce e pubblica. I push su altri branch generano **deploy preview** con URL temporanei: utilissimi per testare prima del merge.

---

## 10. Dominio personalizzato e HTTPS

### 10.1 Scegli e acquista un dominio

Registrar consigliati:

- **Cloudflare Registrar** — prezzo di costo, **il migliore**
- **Namecheap** — interfaccia semplice, prezzi onesti
- **Porkbun** — economico, ottima UX

Costo medio: **10-15 €/anno** per `.it` o `.com`.

**Consiglio SEO**: scegli un dominio breve, memorabile, possibilmente con la keyword principale o il nome del brand. Evita trattini multipli e numeri.

### 10.2 Collega il dominio a Netlify

1. Su Netlify: `Domain settings → Add a domain`
2. Inserisci `tuosito.com`
3. Netlify ti darà **4 nameserver** (es. `dns1.p01.nsone.net`, ecc.)
4. Vai sul pannello del registrar → imposta i nameserver di Netlify

In alternativa, mantieni i nameserver del registrar e configura solo i record `A` e `CNAME` come Netlify suggerisce.

### 10.3 HTTPS automatico

Netlify attiva **Let's Encrypt** automaticamente entro qualche ora. Una volta attivo, vai in `Domain settings → HTTPS → Force HTTPS` per redirigere tutto il traffico HTTP a HTTPS.

---

## 11. SEO on-page e dati strutturati

Questa è la sezione più importante se vuoi che il sito venga trovato su Google.

### 11.1 Componente SEO riutilizzabile

Crea `src/components/SEO.astro`:

```astro
---
interface Props {
  title: string;
  description: string;
  image?: string;
  canonical?: string;
  noindex?: boolean;
  type?: 'website' | 'article';
  publishedTime?: string;
  modifiedTime?: string;
}

const {
  title,
  description,
  image = '/og/default.jpg',
  canonical,
  noindex = false,
  type = 'website',
  publishedTime,
  modifiedTime,
} = Astro.props;

const siteUrl = 'https://www.tuosito.com';
const canonicalURL = new URL(canonical || Astro.url.pathname, siteUrl);
const imageURL = new URL(image, siteUrl);
---

<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>{title}</title>
<meta name="description" content={description} />
<link rel="canonical" href={canonicalURL} />
{noindex && <meta name="robots" content="noindex, nofollow" />}

<!-- Open Graph -->
<meta property="og:type" content={type} />
<meta property="og:title" content={title} />
<meta property="og:description" content={description} />
<meta property="og:image" content={imageURL} />
<meta property="og:url" content={canonicalURL} />
<meta property="og:locale" content="it_IT" />
<meta property="og:site_name" content="Studio Rossi" />
{publishedTime && <meta property="article:published_time" content={publishedTime} />}
{modifiedTime && <meta property="article:modified_time" content={modifiedTime} />}

<!-- Twitter/X -->
<meta name="twitter:card" content="summary_large_image" />
<meta name="twitter:title" content={title} />
<meta name="twitter:description" content={description} />
<meta name="twitter:image" content={imageURL} />

<!-- Favicon -->
<link rel="icon" type="image/svg+xml" href="/favicon.svg" />
<link rel="apple-touch-icon" href="/apple-touch-icon.png" />

<!-- Performance hints -->
<link rel="preconnect" href="https://plausible.io" />
```

Usalo in ogni pagina:

```astro
<SEO
  title="Studio Rossi — Architettura sostenibile a Milano"
  description="Progettiamo edifici a basso impatto ambientale dal 2010. Bioedilizia, ristrutturazioni green, consulenze CasaClima."
/>
```

### 11.2 Regole per `<title>` e `<meta description>` perfetti

- **Title** — 50-60 caratteri. Formula: `Keyword principale | Brand`
- **Description** — 140-160 caratteri. Riassume la pagina e invita al click (CTA implicita)
- **Ogni pagina deve avere title e description UNICI**

### 11.3 `robots.txt`

Crea `public/robots.txt`:

```
User-agent: *
Allow: /

# Blocca pagine non utili agli indici
Disallow: /admin/
Disallow: /api/

Sitemap: https://www.tuosito.com/sitemap-index.xml
```

### 11.4 Registra il sito su Google Search Console e Bing Webmaster

- [search.google.com/search-console](https://search.google.com/search-console) — aggiungi la proprietà del dominio, verifica tramite DNS (Netlify ha l'opzione integrata), invia la sitemap
- [bing.com/webmasters](https://www.bing.com/webmasters) — Bing alimenta anche Copilot e DuckDuckGo, non è più trascurabile nel 2026

### 11.5 Dati strutturati Schema.org

Per un'attività locale, aggiungi nel `<head>` della home:

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "LocalBusiness",
  "name": "Studio Rossi Architetti",
  "image": "https://www.tuosito.com/og/studio.jpg",
  "url": "https://www.tuosito.com",
  "telephone": "+39 02 1234567",
  "email": "info@tuosito.com",
  "address": {
    "@type": "PostalAddress",
    "streetAddress": "Via Roma 1",
    "addressLocality": "Milano",
    "postalCode": "20100",
    "addressRegion": "MI",
    "addressCountry": "IT"
  },
  "geo": {
    "@type": "GeoCoordinates",
    "latitude": 45.4642,
    "longitude": 9.1900
  },
  "openingHoursSpecification": [{
    "@type": "OpeningHoursSpecification",
    "dayOfWeek": ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday"],
    "opens": "09:00",
    "closes": "18:00"
  }],
  "sameAs": [
    "https://www.linkedin.com/company/studio-rossi",
    "https://www.instagram.com/studiorossi"
  ]
}
</script>
```

Per gli articoli del blog usa `@type: "Article"`, per la home aziendale `Organization`, per le FAQ `FAQPage`. Testa con [validator.schema.org](https://validator.schema.org) e [search.google.com/test/rich-results](https://search.google.com/test/rich-results).

### 11.6 URL puliti

Astro genera URL puliti per default. Regole:

- Minuscoli: `/servizi/consulenza-energetica` non `/Servizi/ConsulenzaEnergetica`
- Trattini per separare le parole, mai underscore
- Brevi ma descrittivi
- Strutturali e gerarchici: `/servizi/<nome-servizio>` invece di `/pagina-12345`

### 11.7 Internal linking

Collega le pagine tra loro con anchor descrittivi. Da ogni pagina di servizio linka:

- Altri servizi correlati
- La pagina contatti
- Casi studio relativi

Questo aiuta Google a capire la struttura del sito e migliora il tempo di permanenza.

---

## 12. Accessibilità (WCAG)

L'accessibilità non è opzionale: è **fattore di ranking SEO**, è **richiesta dalla legge** in UE (European Accessibility Act in vigore dal 28 giugno 2025), e amplia il pubblico raggiungibile.

### 12.1 Linee guida WCAG 2.2 livello AA

Obiettivo minimo per un sito vetrina professionale:

- **Contrasto testo/sfondo** almeno 4.5:1 per il testo normale, 3:1 per il testo grande
- **Tutti gli elementi interattivi** raggiungibili con tastiera (`Tab`, `Enter`, `Esc`)
- **Indicatore di focus visibile** (non rimuovere `outline` senza sostituirlo)
- **Alt text descrittivo** per ogni immagine significativa, `alt=""` per le decorative
- **Struttura semantica**: usa `<header>`, `<main>`, `<nav>`, `<footer>`, gerarchia `h1 → h2 → h3` corretta
- **`lang` dichiarato**: `<html lang="it">`
- **Form**: ogni `<input>` ha la sua `<label>` collegata via `for`/`id`
- **Movimento ridotto**: rispetta `prefers-reduced-motion`

### 12.2 Esempio CSS per focus accessibile

```css
*:focus-visible {
  outline: 3px solid #1d4ed8;
  outline-offset: 2px;
  border-radius: 2px;
}

@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

### 12.3 Strumenti di test

- **axe DevTools** (estensione Chrome/Firefox) — scansione one-click
- **Lighthouse** → tab Accessibility (incluso in Chrome DevTools)
- **WAVE** ([wave.webaim.org](https://wave.webaim.org)) — analisi visiva
- **Test manuale** con tastiera: chiudi il mouse, naviga tutto il sito solo con `Tab`

Obiettivo: **Lighthouse Accessibility ≥ 95**.

---

## 13. Analytics e monitoraggio

### 13.1 Analytics rispettosi della privacy (consigliato)

| Servizio | Costo | Caratteristiche |
|---|---|---|
| **Plausible** | da 9 $/mese | Cloud-hosted, GDPR senza cookie banner |
| **Umami** | gratis (self-hosted) | Open source, installabile gratuitamente su Vercel/Netlify |
| **Netlify Analytics** | 9 $/mese | Lato server, nessun JavaScript nel browser |
| **Pirsch** | da 6 $/mese | Alternativa europea a Plausible |

**Sconsigliato per siti vetrina**: Google Analytics 4. Richiede cookie banner, è invasivo, e nel 2026 la sua compatibilità con il GDPR resta controversa in diversi paesi UE.

### 13.2 Installare Plausible

Aggiungi nel `<head>` (o nel componente `SEO.astro`):

```html
<script defer data-domain="tuosito.com" src="https://plausible.io/js/script.js"></script>
```

Nessun cookie, nessun consenso richiesto.

### 13.3 Monitoraggio uptime

- **UptimeRobot** ([uptimerobot.com](https://uptimerobot.com)) — gratuito fino a 50 monitor, controlli ogni 5 minuti, avvisi via email/Telegram
- **Better Stack** — alternativa più moderna, piano gratuito generoso

### 13.4 Error tracking

Per essere avvisato quando qualcuno incontra un errore JavaScript:

- **Sentry** ([sentry.io](https://sentry.io)) — gratuito fino a 5.000 eventi/mese, integrazione Astro ufficiale
- **GlitchTip** — alternativa open source compatibile con l'SDK di Sentry

---

## 14. Performance e Core Web Vitals

Le prestazioni sono **fattore di ranking confermato da Google** dal 2021. Le Core Web Vitals 2026 sono:

- **LCP** (Largest Contentful Paint) — quando appare l'elemento principale → obiettivo **< 2,5 s**
- **CLS** (Cumulative Layout Shift) — quanto "salta" il layout durante il caricamento → obiettivo **< 0,1**
- **INP** (Interaction to Next Paint) — reattività agli input → obiettivo **< 200 ms**

### 14.1 Immagini

Le immagini sono la prima causa di siti lenti. Regole:

- **Sempre WebP o AVIF** (Astro lo fa automaticamente con il componente `<Image />`)
- Mai PNG o JPG sopra i 200 KB
- **Lazy loading** per le immagini sotto la piega: `loading="lazy"`
- **Eager loading** per l'immagine LCP (di solito quella hero): `loading="eager"` + `fetchpriority="high"`
- **Width e height espliciti** per evitare CLS

```astro
---
import { Image } from 'astro:assets';
import hero from '../assets/hero.jpg';
---
<Image
  src={hero}
  alt="Foto dello studio di architettura Rossi a Milano"
  loading="eager"
  fetchpriority="high"
  widths={[400, 800, 1200, 1600]}
  sizes="(max-width: 768px) 100vw, 1200px"
  format="avif"
/>
```

Per compressione manuale extra: [squoosh.app](https://squoosh.app).

### 14.2 Font

- Preferisci **font di sistema** quando possibile (zero peso, massima velocità)
- Se usi Google Fonts, **self-hosting** con il pacchetto `@fontsource/<nome-font>` (privacy + velocità)
- Usa sempre `font-display: swap` per evitare il "flash of invisible text"
- **Preload** del font del titolo

### 14.3 JavaScript

Astro non spedisce JavaScript di default. Quando ti serve interattività usa la direttiva di hydration più leggera possibile:

```astro
<Counter client:visible />   <!-- carica solo quando il componente entra nel viewport -->
<Counter client:idle />      <!-- carica quando il browser è in idle -->
<Counter client:load />      <!-- carica subito (sconsigliato a meno che serva davvero) -->
```

### 14.4 Strumenti di test

- **PageSpeed Insights** ([pagespeed.web.dev](https://pagespeed.web.dev)) — obiettivo **90+ su mobile**
- **GTmetrix** ([gtmetrix.com](https://gtmetrix.com))
- **WebPageTest** ([webpagetest.org](https://webpagetest.org)) — il più accurato

---

## 15. Form di contatto

### 15.1 Netlify Forms

Aggiungi `data-netlify="true"` al tuo `<form>`. Esempio completo accessibile e anti-spam:

```html
<form
  name="contatti"
  method="POST"
  data-netlify="true"
  data-netlify-honeypot="bot-field"
  action="/grazie"
>
  <p hidden>
    <label>Non riempire questo campo: <input name="bot-field" /></label>
  </p>

  <label for="nome">Nome*</label>
  <input id="nome" type="text" name="nome" required autocomplete="name" />

  <label for="email">Email*</label>
  <input id="email" type="email" name="email" required autocomplete="email" />

  <label for="telefono">Telefono</label>
  <input id="telefono" type="tel" name="telefono" autocomplete="tel" />

  <label for="messaggio">Messaggio*</label>
  <textarea id="messaggio" name="messaggio" required rows="5"></textarea>

  <label>
    <input type="checkbox" name="privacy" required />
    Ho letto e accetto la <a href="/privacy">privacy policy</a>
  </label>

  <button type="submit">Invia</button>
</form>
```

> **Nota tecnica**: con Astro che genera HTML statico, l'`<input type="hidden" name="form-name">` **non è necessario** (lo è solo per form renderizzati lato client con framework JS).

### 15.2 Notifiche email

Su Netlify: `Forms → Notifications → Add notification → Email`. Inserisci la tua email e riceverai ogni nuovo invio.

### 15.3 Protezione anti-spam

In ordine crescente di severità:

1. **Honeypot** (già attivo con `netlify-honeypot`) — basta per il 90% dei casi
2. **reCAPTCHA invisibile** o **hCaptcha** — aggiungi `data-netlify-recaptcha="true"`
3. **Akismet** — filtro AI (opzione a pagamento)

### 15.4 Crea una pagina di ringraziamento (`/grazie`)

Permette di tracciare le **conversioni** in analytics e migliora l'UX.

---

## 16. Sicurezza, CSP e privacy

### 16.1 Variabili d'ambiente

**Non committare mai** chiavi API, password o token. Usa file `.env`:

```
PUBLIC_PLAUSIBLE_DOMAIN=tuosito.com
SECRET_NEWSLETTER_API_KEY=xxxxx
```

- Variabili che iniziano con `PUBLIC_` sono accessibili nel browser
- Le altre sono **solo build-time**, mai esposte al client
- Crea sempre `.env.example` (senza valori reali) da committare

Su Netlify: `Site settings → Environment variables` per le chiavi di produzione.

### 16.2 Dipendenze sempre aggiornate

```bash
npm outdated         # Vedi cosa è obsoleto
npm update           # Aggiorna le minor/patch
npm audit            # Controlla vulnerabilità note
npm audit fix        # Risolvi automaticamente quando possibile
```

Attiva **Dependabot** su GitHub: `Settings → Code security → Enable Dependabot security updates`. Riceverai PR automatiche quando esce un fix di sicurezza.

### 16.3 Content Security Policy

La CSP è la difesa più importante contro gli attacchi XSS. La trovi già nel `netlify.toml` della sezione 9.4 — adattala in base ai servizi esterni che usi (analytics, mappe, embed video). Testala con [csp-evaluator.withgoogle.com](https://csp-evaluator.withgoogle.com).

### 16.4 Pagine legali obbligatorie (UE/Italia)

- **Privacy policy** — sempre obbligatoria se raccogli email o anche solo log di accesso
- **Cookie policy** — se usi cookie non strettamente tecnici
- **Termini di servizio** — consigliata anche per un vetrina

Genera privacy e cookie policy con:

- **Iubenda** ([iubenda.com](https://www.iubenda.com)) — ~30 €/anno, italiano nativo
- **Cookiebot** — alternativa nordica molto solida
- **Klaro!** — soluzione open source per il consent banner

### 16.5 Consent banner

Se usi **solo cookie tecnici e analytics privacy-friendly** (Plausible, Umami): non serve banner — basta la privacy policy.

Se usi **Google Analytics, Meta Pixel, Hotjar o simili**: serve un consent banner con scelta granulare (accetta tutti / rifiuta tutti / personalizza), e gli script di tracciamento devono caricarsi **solo dopo il consenso**.

---

## 17. CMS opzionale per non sviluppatori

Se il tuo cliente (o tu stesso) vuoi modificare i contenuti senza toccare il codice:

### Decap CMS (gratuito, open source)

Ex Netlify CMS. Si integra direttamente con GitHub: il cliente apre `/admin`, modifica i testi in un'interfaccia visuale, e i cambiamenti diventano commit Git.

```bash
npm install decap-cms-app
```

Pro: gratuito, dati nel tuo repo, niente database. Contro: setup iniziale un po' macchinoso.

### Sanity

CMS headless cloud. Piano gratuito generoso. Più potente di Decap, perfetto se i contenuti sono complessi (blog, gallerie, campi strutturati).

### Storyblok

Visual editing in tempo reale. Piano gratuito. Ottimo per agenzie con clienti non tecnici.

Per un sito vetrina classico di 5-7 pagine, di solito **non serve un CMS**: bastano file Markdown nel repository.

---

## 18. CI/CD con GitHub Actions

Automatizza lint, build e test su ogni Pull Request. Crea `.github/workflows/ci.yml`:

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'

      - run: npm ci

      - name: Lint
        run: npm run lint --if-present

      - name: Format check
        run: npx prettier --check .

      - name: Type check
        run: npx astro check

      - name: Build
        run: npm run build
```

Ad ogni push, GitHub esegue questi controlli **prima** che Netlify pubblichi: se uno fallisce, lo scopri subito invece di rompere la produzione.

---

## 19. Test automatici end-to-end

Anche un sito vetrina merita uno **smoke test** che verifichi che le pagine principali rispondano correttamente.

### Setup Playwright

```bash
npm init playwright@latest
```

Crea `tests/home.spec.ts`:

```ts
import { test, expect } from '@playwright/test';

test('la home carica e contiene il titolo', async ({ page }) => {
  await page.goto('/');
  await expect(page).toHaveTitle(/Studio Rossi/);
  await expect(page.getByRole('heading', { level: 1 })).toBeVisible();
});

test('la pagina contatti ha il form', async ({ page }) => {
  await page.goto('/contatti');
  await expect(page.getByLabel('Email')).toBeVisible();
  await expect(page.getByRole('button', { name: /invia/i })).toBeVisible();
});

test('nessun errore console su home', async ({ page }) => {
  const errors: string[] = [];
  page.on('pageerror', (e) => errors.push(e.message));
  await page.goto('/');
  expect(errors).toEqual([]);
});
```

Aggiungi al workflow GitHub Actions:

```yaml
- name: Install Playwright browsers
  run: npx playwright install --with-deps

- name: Run E2E tests
  run: npx playwright test
```

---

## 20. `llms.txt`: ottimizzazione per motori AI

Nel 2026, ChatGPT, Claude, Perplexity e i motori AI sono fonti di traffico significative. Il file `llms.txt` è lo standard emergente per dichiarare a questi motori cosa indicizzare e come.

Crea `public/llms.txt`:

```
# Studio Rossi Architetti

> Studio di architettura sostenibile con sede a Milano. Specializzati in
> bioedilizia, ristrutturazioni green e consulenze CasaClima dal 2010.

## Servizi
- [Progettazione bioedilizia](https://www.tuosito.com/servizi/bioedilizia)
- [Ristrutturazioni sostenibili](https://www.tuosito.com/servizi/ristrutturazioni)
- [Consulenza CasaClima](https://www.tuosito.com/servizi/casaclima)

## Informazioni
- [Chi siamo](https://www.tuosito.com/chi-siamo)
- [Portfolio](https://www.tuosito.com/portfolio)
- [Contatti](https://www.tuosito.com/contatti)

## Optional
- [Blog](https://www.tuosito.com/blog)
```

Standard ufficiale: [llmstxt.org](https://llmstxt.org).

---

## 21. Checklist pre-lancio

Spunta tutto prima di considerare il sito "pubblico":

### Contenuti

- [ ] Tutte le pagine hanno `<title>` e `<meta description>` unici e ottimizzati
- [ ] Open Graph testato con [opengraph.xyz](https://www.opengraph.xyz)
- [ ] Tutte le immagini hanno `alt` descrittivo (o `alt=""` se decorative)
- [ ] Niente errori di battitura (passa Hemingway o LanguageTool)
- [ ] Pagine legali (privacy, cookie) presenti e linkate nel footer

### Tecnica

- [ ] Sito responsive testato su mobile (Chrome DevTools → Toggle device)
- [ ] Test su Chrome, Firefox, Safari, Edge
- [ ] Form di contatto funzionante (invia un test reale)
- [ ] HTTPS attivo e forzato
- [ ] `www` e root puntano alla stessa versione (uno reindirizza all'altro)
- [ ] Pagina 404 personalizzata
- [ ] Favicon caricata (SVG + Apple touch icon)

### SEO

- [ ] Sitemap raggiungibile su `/sitemap-index.xml`
- [ ] `robots.txt` corretto
- [ ] Sito registrato su Google Search Console
- [ ] Sito registrato su Bing Webmaster Tools
- [ ] Dati strutturati validati con [search.google.com/test/rich-results](https://search.google.com/test/rich-results)
- [ ] `llms.txt` presente

### Performance

- [ ] PageSpeed Insights **mobile ≥ 90**
- [ ] LCP < 2,5 s, CLS < 0,1, INP < 200 ms
- [ ] Tutte le immagini in WebP/AVIF
- [ ] Lazy loading attivo per le immagini sotto la piega

### Accessibilità

- [ ] Lighthouse Accessibility **≥ 95**
- [ ] Navigazione completa con tastiera
- [ ] Contrasto minimo 4.5:1 verificato

### Sicurezza

- [ ] Security headers attivi (testa su [securityheaders.com](https://securityheaders.com))
- [ ] CSP configurata e testata
- [ ] Dependabot attivo
- [ ] Nessun secret committato (controlla con `git log --all -p | grep -i secret`)

### Link rotti

- [ ] Controllo con [deadlinkchecker.com](https://www.deadlinkchecker.com)

---

## 22. Manutenzione continua

### Settimanale

- Controlla nuovi form di contatto su Netlify
- Rispondi alle email
- Verifica gli analytics: cosa funziona, cosa no

### Mensile

```bash
npm outdated
npm update
npm audit fix
```

- Lighthouse audit rapido
- Revisione metriche Search Console: query, click, impression, posizione media

### Trimestrale

- Aggiornamento contenuti (foto, news, casi studio)
- Revisione SEO: parole chiave, contenuti da aggiungere
- Verifica backup (GitHub fa già da backup; verifica solo che il push funzioni)

### Annuale

- Rinnovo dominio (verifica che l'autorinnovo sia attivo)
- Revisione completa del sito
- Aggiorna Node.js LTS se è uscita una major nuova
- Audit di sicurezza completo

---

## 23. Risoluzione problemi (FAQ)

### Quanto costa davvero gestire un sito vetrina con questo stack?

Il **dominio** costa 10-15 €/anno. L'**hosting su Netlify** è gratuito (piano Starter). Se aggiungi **Plausible Analytics** sono altri ~108 $/anno. Totale: **0-130 € all'anno**. Niente plugin, niente licenze, niente sorprese.

### Quanto tempo serve per pubblicare il primo sito?

Seguendo questa guida senza interruzioni: **un pomeriggio** (4-6 ore) per la prima messa online, qualche giorno in più per scrivere i contenuti e perfezionare il design.

### Devo saper programmare per usare Astro?

Aiuta sapere le basi di HTML e CSS. Tutto il resto si impara strada facendo. Se non hai mai scritto codice, puoi cominciare modificando il template e progredire un passo alla volta.

### Posso usare WordPress invece di Astro?

Sì, ma per un sito vetrina è una scelta peggiore nel 2026: WordPress richiede manutenzione costante, plugin di sicurezza, hosting a pagamento (15-30 €/mese minimo), e il sito risulterà più lento. Astro produce siti più veloci, più sicuri, e a costo praticamente zero.

### Il dev server non parte: cosa faccio?

Cancella `node_modules` e reinstalla:

- **macOS/Linux**: `rm -rf node_modules package-lock.json && npm install`
- **Windows (PowerShell)**: `Remove-Item -Recurse -Force node_modules, package-lock.json; npm install`
- **Windows (CMD)**: `rmdir /s /q node_modules & del package-lock.json & npm install`

### Il build fallisce su Netlify ma funziona in locale

Quasi sempre è uno di tre motivi: la **versione di Node** è diversa (specifica `NODE_VERSION = "22"` in `netlify.toml`), **variabili d'ambiente** mancanti (configura su Netlify → Site settings → Environment variables), o **dipendenze non committate** (controlla che `package-lock.json` sia nel repo).

### Le modifiche non appaiono online

`Ctrl+Shift+R` per un hard refresh. Se non basta, controlla che il deploy su Netlify sia andato a buon fine: `Deploys → Last deploy → Published`. Se è fallito, leggi i log.

### Il form Netlify non riceve invii

Verifica tre cose: 1) l'attributo `data-netlify="true"` è presente sul `<form>`; 2) l'attributo `name="contatti"` (o quello che hai scelto) è presente; 3) il form appare in `Forms` nella dashboard Netlify. Se è la prima volta, ti serve un primo invio "vero" per attivare la form detection.

### Quanto traffico può reggere il piano gratuito di Netlify?

**100 GB di banda al mese** e 300 minuti di build. Per un sito vetrina sono migliaia di visitatori al giorno senza problemi.

### Astro è davvero meglio di Next.js per un sito vetrina?

Sì, **per un sito vetrina**. Astro genera HTML puro senza JavaScript: caricamento istantaneo, SEO perfetto, manutenzione minima. Next.js è ottimo ma è progettato per applicazioni web interattive e ne paghi il prezzo in complessità e peso del bundle.

### Come faccio a comparire su Google?

Tre cose, in ordine: 1) registra il sito su **Google Search Console** e invia la sitemap; 2) ottieni **link in entrata** da siti rilevanti del tuo settore (directory locali, partnership, social); 3) scrivi **contenuti utili e specifici** (un blog con articoli sulle domande dei tuoi clienti aiuta moltissimo). La SEO richiede pazienza: i primi risultati si vedono in 1-3 mesi.

### Quale CMS scegliere se voglio modificare i contenuti senza toccare il codice?

Per un sito statico Astro: **Decap CMS** (gratuito, dati nel tuo repo Git) per esigenze semplici, **Sanity** o **Storyblok** se hai contenuti complessi e clienti non tecnici. Per tre pagine fisse non serve un CMS.

---

## 24. Risorse utili

### Documentazione ufficiale

- [Documentazione Astro](https://docs.astro.build)
- [Documentazione Tailwind CSS](https://tailwindcss.com/docs)
- [Documentazione Netlify](https://docs.netlify.com)
- [MDN Web Docs (in italiano)](https://developer.mozilla.org/it)

### Approfondimenti

- [Pro Git (libro gratuito in italiano)](https://git-scm.com/book/it/v2)
- [web.dev — Best practice Google](https://web.dev)
- [Can I Use — Compatibilità browser](https://caniuse.com)
- [WCAG 2.2 Quick Reference](https://www.w3.org/WAI/WCAG22/quickref/)

### Strumenti SEO

- [Google Search Console](https://search.google.com/search-console)
- [Bing Webmaster Tools](https://www.bing.com/webmasters)
- [Schema.org Validator](https://validator.schema.org)
- [Rich Results Test](https://search.google.com/test/rich-results)

### Strumenti di test

- [PageSpeed Insights](https://pagespeed.web.dev)
- [GTmetrix](https://gtmetrix.com)
- [WebPageTest](https://webpagetest.org)
- [Security Headers Checker](https://securityheaders.com)
- [CSP Evaluator](https://csp-evaluator.withgoogle.com)
- [axe DevTools](https://www.deque.com/axe/devtools/)
- [WAVE Accessibility](https://wave.webaim.org)

---

## Conclusione

Seguendo questa guida nell'ordine hai un **sito vetrina di livello agenzia**: veloce sotto i 2 secondi, perfettamente ottimizzato per SEO, accessibile, sicuro, gratuito da ospitare, e mantenibile con poco sforzo.

Le scelte tecniche (Astro 5, Tailwind 4, Netlify, GitHub) sono **gli standard professionali del 2026** e ti accompagneranno per anni senza dover rifare tutto da capo.

**Prossimi passi consigliati**: completa la checklist pre-lancio, registra il sito su Search Console, scrivi i primi articoli del blog (se hai scelto di averne uno) e crea un calendario di manutenzione mensile. Il sito è solo l'inizio: la SEO e i contenuti sono il lavoro continuo che farà la differenza.

Buon lavoro.

---

*Hai trovato utile questa guida? Condividila con chi sta partendo. Hai domande, errori da segnalare o suggerimenti? Apri una Issue sul repository GitHub o scrivimi a [tua@email.com](mailto:tua@email.com).*

---

<!--
JSON-LD HowTo schema da incollare nel <head> della pagina che ospita questa guida.
Aiuta Google a generare un rich snippet con i passaggi.
-->

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "HowTo",
  "name": "Come creare un sito vetrina professionale con Astro, GitHub e Netlify",
  "description": "Guida passo-passo per creare un sito vetrina veloce, SEO-friendly e gratuito nel 2026.",
  "totalTime": "PT6H",
  "estimatedCost": {
    "@type": "MonetaryAmount",
    "currency": "EUR",
    "value": "15"
  },
  "supply": [
    { "@type": "HowToSupply", "name": "Computer con Windows, macOS o Linux" },
    { "@type": "HowToSupply", "name": "Connessione internet" },
    { "@type": "HowToSupply", "name": "Email valida" }
  ],
  "tool": [
    { "@type": "HowToTool", "name": "Node.js 22 LTS" },
    { "@type": "HowToTool", "name": "Git" },
    { "@type": "HowToTool", "name": "Visual Studio Code" }
  ],
  "step": [
    { "@type": "HowToStep", "name": "Installa Node.js, Git e VS Code", "url": "#2-strumenti-da-installare" },
    { "@type": "HowToStep", "name": "Configura gli account GitHub e Netlify", "url": "#3-configurazione-account" },
    { "@type": "HowToStep", "name": "Crea il progetto Astro 5", "url": "#4-creazione-del-progetto-astro-5" },
    { "@type": "HowToStep", "name": "Pubblica su GitHub", "url": "#8-pubblicazione-su-github" },
    { "@type": "HowToStep", "name": "Esegui il deploy su Netlify", "url": "#9-deploy-su-netlify" },
    { "@type": "HowToStep", "name": "Collega un dominio personalizzato", "url": "#10-dominio-personalizzato-e-https" },
    { "@type": "HowToStep", "name": "Ottimizza per SEO e accessibilità", "url": "#11-seo-on-page-e-dati-strutturati" },
    { "@type": "HowToStep", "name": "Completa la checklist pre-lancio", "url": "#21-checklist-pre-lancio" }
  ]
}
</script>
```
