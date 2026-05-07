# Miniguida: scaricare video YouTube con yt-dlp su Linux

## 1. Installazione (una sola volta)

```bash
sudo apt install pipx ffmpeg -y
pipx install yt-dlp
pipx ensurepath
```

Chiudi e riapri il terminale.

Verifica che funzioni:

```bash
yt-dlp --version
```

---

## 2. Comando base

```bash
yt-dlp "URL"
```

**Regole d'oro:**
- URL **sempre tra virgolette doppie** `"..."`
- **Niente backslash** prima di `?` `=` `&`
- Il file si salva nella cartella corrente

Esempio corretto:

```bash
yt-dlp "https://www.youtube.com/watch?v=XagPX36sBhI"
```

Esempio **sbagliato** (causa errore 404):

```bash
yt-dlp "https://www.youtube.com/watch\?v\=XagPX36sBhI"
```

---

## 3. Comandi più usati

### Scegliere dove salvare

```bash
yt-dlp -P ~/Video "URL"
```

### Solo audio (MP3)

```bash
yt-dlp -x --audio-format mp3 "URL"
```

### Qualità massima fino a 1080p

```bash
yt-dlp -f "bv*[height<=1080]+ba/b[height<=1080]" "URL"
```

### Vedere i formati disponibili

```bash
yt-dlp -F "URL"
```

### Scaricare un formato specifico

(usando gli ID dalla lista del comando precedente)

```bash
yt-dlp -f 137+140 "URL"
```

### Intera playlist

```bash
yt-dlp -o "%(playlist_index)s - %(title)s.%(ext)s" "URL_PLAYLIST"
```

### Sottotitoli (anche automatici, in italiano)

```bash
yt-dlp --write-subs --write-auto-subs --sub-lang it "URL"
```

### Solo file audio + miniatura + metadati (ottimo per musica)

```bash
yt-dlp -x --audio-format mp3 --embed-thumbnail --add-metadata "URL"
```

---

## 4. Aggiornamento

YouTube cambia spesso, quindi yt-dlp va aggiornato regolarmente:

```bash
pipx upgrade yt-dlp
```

---

## 5. Errori comuni

| Errore | Causa | Soluzione |
|---|---|---|
| `command not found: yt-dlp` | PATH non aggiornato | `pipx ensurepath` + riapri terminale |
| `HTTP Error 404` | URL con `\?` o `\=` | togli i backslash |
| `ffmpeg not found` | manca ffmpeg | `sudo apt install ffmpeg` |
| `Sign in to confirm...` | video con restrizioni d'età | `--cookies-from-browser firefox` |
| `Video unavailable` | restrizione geografica | usa una VPN |
| Download lento | rate limit di YouTube | aggiungi `--limit-rate 5M` o usa cookies |

---

## 6. Tip pratici

### Alias per scaricare sempre nella stessa cartella

Aggiungi al tuo `~/.zshrc` (o `~/.bashrc`):

```bash
echo 'alias ytd="yt-dlp -P ~/Video"' >> ~/.zshrc
source ~/.zshrc
```

Poi basta:

```bash
ytd "URL"
```

### Alias per scaricare solo audio MP3

```bash
echo 'alias ytmp3="yt-dlp -x --audio-format mp3 -P ~/Musica"' >> ~/.zshrc
source ~/.zshrc
```

Uso:

```bash
ytmp3 "URL"
```

### Nome file personalizzato

```bash
yt-dlp -o "%(title)s.%(ext)s" "URL"
```

Variabili utili: `%(title)s`, `%(uploader)s`, `%(upload_date)s`, `%(id)s`, `%(ext)s`.

---

## 7. Riferimenti

- Sito ufficiale: <https://github.com/yt-dlp/yt-dlp>
- Documentazione completa: `man yt-dlp` oppure `yt-dlp --help`

---

> **Nota legale:** scarica solo contenuti per cui hai i diritti, contenuti tuoi, o materiale liberamente distribuibile (es. Creative Commons), nel rispetto dei Termini di Servizio della piattaforma.
