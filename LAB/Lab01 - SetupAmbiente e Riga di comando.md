# Laboratorio di Sicurezza Informatica — Setup Ambiente e Riga di Comando

**Docente:** Prof. Marco Prandini
**Argomento:** Introduzione all'ambiente per le esercitazioni e richiami sull'uso della riga di comando

---

## 1. L'Ambiente di Esercitazione (Virtual Machine)

Per le esercitazioni pratiche si utilizzerà una **macchina virtuale (VM)** basata su una distribuzione Linux (**Parrot OS** o **Kali Linux**) preconfigurata per la offensive security.

### Specifiche e Creazione

- **Requisiti:** La VM è pesante (circa **60 GB**) poiché contiene già tutti gli strumenti del corso.
- **VirtualBox:** Il software di virtualizzazione da utilizzare è **Oracle VM VirtualBox**.
- **Parametri VM:** 8 GB di RAM, 4 core dedicati, 128 MB di memoria video.
- **Rete:** Configurazione di una scheda di rete aggiuntiva (scheda 2) in modalità **"host-only"** connessa a `vboxnet0`.

### Gestione in Laboratorio (LAB4)

Sui PC del laboratorio è obbligatorio usare le proprie **credenziali di Ingegneria**. Poiché il disco `.vdi` di base è condiviso e protetto da scrittura, è indispensabile creare uno **Snapshot** prima di avviare la macchina.

Gli snapshot permettono di salvare lo stato della VM e scrivere le modifiche in un file separato nella cartella personale dell'utente, senza alterare l'immagine originale.

> Il salvataggio va indirizzato nella cartella `~/large/` per evitare limiti di spazio.

---

## 2. Accesso e Sicurezza di Base

- **Credenziali VM:** Utente `parrot`, password `parrot` (oppure `kali` / `kali`).
- **Privilegi Amministratore:** I comandi che richiedono privilegi elevati si eseguono anteponendo `sudo` (es. `sudo cat /etc/shadow`).

### Gestione Password e "Salt"

Il sistema non memorizza le password in chiaro, ma ne conserva solo un'**impronta (hash)** per impedire il furto diretto.

- **`/etc/passwd`** — File leggibile da tutti (*world-readable*) che contiene informazioni sugli utenti.
- **`/etc/shadow`** — File accessibile solo a `root` che contiene l'hash delle password.

### Il Problema del Dizionario

Per evitare attacchi con tabelle precalcolate (**Rainbow Tables**) o l'identificazione di password identiche tra più utenti, viene aggiunto un **salt**.

**Salt:** è una stringa casuale concatenata alla password prima del calcolo dell'hash. Cambia a ogni rinnovo della password.

Formato in `/etc/shadow`:

```
$ID_algoritmo$salt$hash
```

---

## 3. Comandi Essenziali e Navigazione

La **shell** (es. `bash`) è l'interfaccia a riga di comando per interagire con il sistema operativo.

### File e Cartelle

- **`pwd`** — Mostra la directory corrente di lavoro.
- **`cd`** — Cambia directory (usa percorsi assoluti o relativi).
- **`ls`** — Elenca i file. Opzioni utili:
  - `-l` — formato lungo con dettagli
  - `-a` — mostra i file nascosti (che iniziano per `.`)
  - `-R` — ricorsivo
  - `-t` — ordina per data

### Timestamps (`stat`)

Ogni file possiede **marcatori temporali**:

- **`mtime`** — modifica del contenuto
- **`atime`** — ultimo accesso
- **`ctime`** — modifica dei metadati

Possono essere visualizzati con `stat` o forzati con `touch`.

### Ricerca nel Filesystem

- **`find`** — Cerca in **tempo reale** esplorando il disco. Molto potente ma genera carico. Permette di cercare per nome, dimensione, timestamp e di eseguire comandi sui risultati tramite l'opzione `-exec`.

- **`locate`** — Cerca **istantaneamente** interrogando un database indicizzato (aggiornato tramite `updatedb`). È veloce ma può fornire risultati obsoleti se i file sono stati appena creati/cancellati.

### Archiviazione e Compressione

- **`tar`** *(Tape Archiver)* — Concatena file preservandone permessi e ownership. Comandi principali:
  - `-c` — crea
  - `-x` — estrai
  - `-t` — lista
  - `-f` — file destinazione

- **Compressione:** `gzip`, `bzip2`, `xz`. Il comando `tar` può invocare direttamente i compressori usando le opzioni `-z`, `-j`, `-J`.

---

## 4. Input/Output, Filtri e Pipeline

I comandi Linux comunicano tramite tre **stream predefiniti**:

- **Standard Input (stdin)** — File descriptor `0`
- **Standard Output (stdout)** — File descriptor `1`
- **Standard Error (stderr)** — File descriptor `2`

### Ridirezione e Pipe

- **`>`** e **`>>`** — Ridirigono lo stdout su un file (rispettivamente sovrascrivendo o in append).
- **`2>`** e **`2>>`** — Ridirigono lo stderr.
- **`|` (Pipe)** — Connette lo stdout di un processo direttamente allo stdin di un altro (es. `ls | sort`).

### Analisi Testo e Filtri

- **`cat` / `tac`** — Mostrano il file dall'inizio alla fine o viceversa.
- **`less`** — Impagina l'output sul terminale permettendo la navigazione interattiva.
- **`head` / `tail`** — Mostrano rispettivamente l'inizio o la fine di un file. Molto utile: `tail -f` mantiene il file aperto e mostra le nuove righe in tempo reale (ideale per i file di **log**).
- **`cut`** — Estrae porzioni di testo per carattere (`-c`) o per campi/colonne delimitati (`-d` delimitatore, `-f` campi).
- **`sort` / `uniq`** — Ordinano le righe e rimuovono/mostrano i duplicati consecutivi.

### Ricerca Testuale con Grep ed Espressioni Regolari

- **`grep` / `egrep`** — Esamina le righe e riproduce quelle che contengono un pattern basato su **Espressioni Regolari (RE)**.
- **RegEx base:**
  - `^` — inizio riga
  - `$` — fine riga
  - `.` — un carattere qualsiasi
  - `*` — zero o più occorrenze

---

## 5. Gestione dei Processi

- **`ps`** e **`top`** — Mostrano lo stato dei processi e il loro **PID** (Process ID).
- **Foreground / Background:** Un comando seguito da `&` viene eseguito in background. I comandi `fg` e `bg` riportano i processi in primo o secondo piano. `jobs` mostra i job correnti della shell.
- **Segnali da tastiera:**
  - `Ctrl+C` → innesca **SIGINT** (interrompe)
  - `Ctrl+Z` → innesca **SIGTSTP** (sospende)

---

## 6. L'Editor VIM (Basi)

Editor testuale **modale** con tre stati principali:

### Command Mode

Il cursore si muove e i tasti eseguono azioni (non si digita testo). Si ritorna da Input Mode premendo `ESC`.

### Input Mode

Inserimento testo. Vi si entra tramite comandi come:

- `i` — insert
- `a` — append
- `o` — nuova riga

### Directive Mode

Riga di comando interna a Vim (si attiva con `:` o `/`).

### Comandi fondamentali

- `:w` — salva
- `:q` — esci
- `:q!` — esci forzando senza salvare
- `dd` — taglia riga
- `yy` — copia riga
- `p` — incolla
