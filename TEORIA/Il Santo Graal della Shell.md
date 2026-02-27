# **🐧 Il Santo Graal: Survival Guide alla Shell Linux**

**Corso:** Laboratorio di Sicurezza Informatica  
**Obiettivo:** Riferimento rapido e avanzato per l'operatività da riga di comando (CLI), gestione processi, manipolazione del testo e VIM.

## **1\. 🌐 Ambiente, Rete e Utenti**

### **Identità e Gestione Utenti**

* whoami: Mostra l'utente corrente.  
* id: Mostra UID, GID e i gruppi a cui appartiene l'utente.  
* who: Mostra chi è attualmente loggato nel sistema.  
* sudo \<comando\>: Esegue un comando con i privilegi di superutente (root).  
* /etc/passwd: File world-readable. Contiene gli utenti del sistema.  
* /etc/shadow: File leggibile solo da root. Contiene gli hash delle password e i **salt** (es. $6$salt$hash).  
* /etc/group: File world-readable. Contiene l'elenco dei gruppi e i relativi membri (utile per privilege escalation).

### **Rete (VM setup tipico)**

* ip a: Mostra la configurazione delle interfacce di rete.  
  * eth0 (es. 10.0.2.15): NAT, usata per uscire su Internet.  
  * eth1 (es. 192.168.56.N): Host-only, usata per parlare con l'host e le altre VM.

### **Trasferimento File (SCP & Cartelle Condivise)**

* **Da Host a VM:** scp file\_su\_host parrot@192.168.56.N:/path\_su\_vm/  
* **Da VM a Host:** scp parrot@192.168.56.N:/path\_su\_vm/file path\_host  
* **Cartelle Condivise VirtualBox:** Se attivate nelle impostazioni della VM con "Auto-mount", vengono montate in /mnt.  
* **Demone SSH:** Per permettere la connessione via rete, assicurarsi che il servizio sia attivo: sudo systemctl enable \--now ssh.service.

### **Gestione Macchina**

* shutdown \-h now: Arresta immediatamente il sistema.  
* shutdown \-r now: Riavvia immediatamente il sistema.

## **2\. 🛠️ Sopravvivenza e Terminal Hacks**

### **Documentare l'Attacco (Reportistica)**

* script \<nome\_file.log\>: Inizia a registrare **tutto** l'input e l'output del terminale in un file. Fondamentale per i report di laboratorio. Digita exit o Ctrl+D per terminare la registrazione.

### **Velocizzare il Workflow**

* history: Mostra la cronologia numerata di tutti i comandi lanciati.  
* Ctrl+R: Apre la ricerca inversa (reverse-i-search). Inizia a digitare parte di un comando vecchio e bash lo ritroverà automaticamente.  
* alias nome='comando': Crea una scorciatoia (es. alias ll='ls \-la'). *Nota: spariscono alla chiusura del terminale se non inseriti nel .bashrc*.  
* Tab (tasto): Autocompletamento magico per percorsi e comandi. Premilo due volte per vedere tutte le opzioni possibili in caso di ambiguità.

### **Manuali e Aiuto (Quando non hai Internet)**

* man \<comando\>: Apre la pagina del manuale del comando. (Usa /pattern per cercare all'interno come in less).  
* man \-k \<keyword\>: Cerca in tutti i manuali una determinata parola chiave (es. man \-k "network").  
* help \<builtin\>: Mostra l'aiuto per i comandi interni della shell (es. help cd).

## **3\. 🗂 Navigazione, Esplorazione e File System**

### **Navigazione Base e ls "Tattico"**

* pwd: Stampa la directory di lavoro corrente.  
* cd \-: Ritorna alla directory in cui ti trovavi precedentemente (fondamentale per saltare tra due path lunghi).  
* cd \~: Ritorna alla home directory dell'utente.  
* ls \-la: Mostra tutti i file (inclusi i nascosti) in formato lungo.  
* ls \-lart: Ordina i file in base al tempo di modifica (-t) in ordine inverso (-r). **Infallibile per trovare i file appena creati/scaricati in fondo alla lista.**

### **Operazioni File (CRUD)**

* cp \<sorgente\> \<destinazione\>: Copia file/cartelle.  
* mv \<sorgente\> \<destinazione\>: Sposta o rinomina file/cartelle.  
* rm \<file\>: Rimuove un file.  
* rm \-rf \<cartella\>: Rimuove forzatamente e ricorsivamente una cartella e tutto il suo contenuto.  
* mkdir \<cartella\>: Crea una nuova directory.  
* file \<nomefile\>: Rivela il **vero** tipo di file leggendo i *magic numbers*, ignorando l'estensione (molto utile se un malware è mascherato da .txt).  
* ln \-s \<target\> \<nome\_link\>: Crea un link simbolico (symlink).

### **Timestamps (I 3 tempi di un file)**

Fondamentali per l'analisi forense e per capire se un file è stato alterato.

* mtime (Modification): Ultima modifica del *contenuto*.  
* atime (Access): Ultimo *accesso* al file.  
* ctime (Change): Ultima modifica dei *metadati* (es. cambio permessi).  
* **Verifica temporale:** stat \--format='%U %a %z' file (Utente, Permessi, ctime).  
* **Alterazione manuale:** touch file (aggiorna i timestamp all'ora attuale).

### **Il coltellino svizzero: find**

Ricerca in *tempo reale* esplorando il disco. Genera carico ma è infallibile.

* find /usr/src \-name '\*.c' \-size \+100k \-print (Cerca file .c maggiori di 100KB).  
* **La mossa letale (-exec):** Esegue un comando su ogni risultato trovato. La sequenza {} indica il file trovato, \\; termina il comando.  
  * *Esempio pro:* Trova file regolari "orfani" (senza utente), modificati meno di 2 giorni fa, e cerca al loro interno la stringa "TXT":  
    find / \-type f \-nouser \-mtime \-2 \-exec grep \-l TXT {} \\;

### **Ricerca Istantanea: locate**

Interroga un database pre-indicizzato (si aggiorna con updatedb). Veloce ma potenzialmente obsoleto se i file sono stati appena creati/cancellati.

### **Lettura Dati Raw: dd**

Permette di clonare dischi o leggere/scrivere dati ignorando il filesystem.

* dd if=\<file\_input\> of=\<file\_output\> bs=\<byte\> count=\<N\> skip=\<N\> seek=\<N\>

## **4\. 🚰 Stream, Pipe e Redirezione (I/O)**

In Linux ogni processo ha 3 canali di comunicazione standard:

1. **stdin (0):** Standard Input.  
2. **stdout (1):** Standard Output.  
3. **stderr (2):** Standard Error.

### **Redirezione**

* \>: Scrive l'output su file (sovrascrive).  
* \>\>: Scrive l'output su file (appende in coda).  
* 2\>: Redirige solo gli errori (stderr).  
* \> file 2\>&1: Redirige sia stdout che stderr nello stesso file.  
* \<: Inietta un file nello stdin di un comando (es. sort \< miofile).

### **Concatenazione (Pipes |)**

Il carattere | connette lo stdout di un processo allo stdin del successivo.

* *Esempio:* ls | sort

## **5\. ✂️ Filtri e Manipolazione del Testo**

* **cat / tac:** Stampa il file dall'inizio alla fine (cat) o dalla fine all'inizio (tac).  
* **less:** Impagina l'output sul terminale. Comandi utili in less:  
  * F: Follow, scorre il file in tempo reale (come tail \-f).  
  * /pattern: Cerca in avanti. ?pattern: Cerca all'indietro. (n per il successivo).  
* **head / tail:** Estrae inizio o fine di un file.  
  * tail \-f file: Mantiene aperto il file per leggere i log in tempo reale.  
* **rev:** Inverte l'ordine dei caratteri di ogni riga.  
* **cut:** Ritaglia colonne/campi.  
  * cut \-c8-30 (dal carattere 8 al 30).  
  * cut \-d: \-f1 (usa i due punti : come delimitatore ed estrae il primo campo).  
* **sort:** Ordina le righe.  
  * \-r (reverse), \-n (numerico), \-u (unico).  
  * **Sort per colonna (Avanzato):** \-t: (imposta i due punti come separatore), \-k3,3n (ordina usando solo la 3a colonna, in formato numerico). Es: sort \-t: \-k3,3n /etc/passwd.  
* **uniq:** Rimuove/mostra duplicati *consecutivi* (usare sempre dopo sort).  
  * \-c (conta le occorrenze), \-d (mostra solo i duplicati).  
* **wc:** Word count. \-l (conta linee), \-c (conta caratteri).

## **6\. 🔍 Regex ed Estrazione Dati: egrep**

L'uso di espressioni regolari estese (egrep o grep \-E) è vitale per estrarre password, IP o dati sensibili dai log.

### **Sintassi Base**

* ^: Inizio riga.  
* $: Fine riga.  
* .: Un carattere qualsiasi.  
* \\w: Lettera, numero o underscore.  
* \[abc\]: 'a', 'b' o 'c'. \[^abc\]: Qualsiasi cosa tranne 'a', 'b' o 'c'.

### **Moltiplicatori (Greediness)**

Le regex sono "ingorde": cercano sempre la corrispondenza più lunga possibile.

* ?: Zero o una occorrenza.  
* \*: Zero o più occorrenze.  
* \+: Una o più occorrenze.  
* {n,m}: Da n a m occorrenze.

### **Opzioni letali di grep**

* \-v: **Inverte** la ricerca (mostra le righe che *non* matchano).  
* \-i: Case-insensitive (ignora maiuscole/minuscole).  
* \-r: Cerca ricorsivamente in tutte le cartelle.  
* \-l: Mostra solo il *nome del file* in cui è stato trovato il match, non la riga.

## **7\. ⚙️ Gestione Processi e Job Control**

* **systemctl:** Gestisce i demoni in background (es. systemctl restart ssh).  
* **top**: Monitoraggio interattivo del sistema (CPU, RAM) e dei processi più attivi.  
* **ps auxw:** Mostra tutti i processi attivi nel sistema con massimo dettaglio.  
* **kill \-\<segnale\> \<PID\>:** Invia segnali ai processi (es. \-9 per terminare brutalmente).  
  * Usa kill \-l per listare tutti i segnali supportati.  
* **sleep \<N\>:** Sospende l'esecuzione per N secondi (fondamentale negli script bash per creare delay).

### **Sopravvivenza nel Terminale (Job Control)**

Se lanci un comando che blocca il terminale (o un exploit che ci mette tanto):

* Ctrl+C: Invia **SIGINT** (interrompe e uccide il processo).  
* Ctrl+Z: Invia **SIGTSTP** (sospende il processo e ti ridà la shell).  
* bg %1: Prende il job 1 sospeso e lo manda in **background**.  
* fg %1: Riporta il job 1 in **foreground** (primo piano).  
* jobs: Elenca tutti i processi avviati dalla shell corrente.  
* **Comando in background:** Aggiungi & alla fine del comando (es. nmap \-sS target &).  
* **Variabile magica:** $\! contiene il PID dell'ultimo processo lanciato in background.

## **8\. 📝 Archiviazione e Compressione**

**tar non comprime di default, archivia solo.** Per comprimere usa i flag specifici.

* tar cvpf archivio.tar /cartella: Crea (c) archivio, verboso (v), preserva permessi (p), su file (f).  
* tar xvpf archivio.tar: Estrae (x) l'archivio.  
* **Compressione al volo:** Aggiungi \-z (gzip, .tar.gz), \-j (bzip2, .tar.bz2), o \-J (xz, .tar.xz).  
  * *Esempio:* tar cvpzf backup.tar.gz /home/user  
* **Decompressione rapida stream:** zcat file.gz o zegrep 'pattern' file.gz.

## **9\. 🧛 VIM: Modalità e Comandi di Sopravvivenza**

Editor modale. Si avvia in **Command Mode**. Premi ESC per tornare in Command Mode da qualsiasi altra modalità.

### **Entrare in Input Mode (Digitare testo)**

* i: Inserisce *prima* del cursore.  
* a: Inserisce *dopo* il cursore (append).  
* A: Inserisce a *fine riga*.  
* o: Apre una *nuova riga sotto*.  
* cw: *Change Word*, cancella la parola corrente e ti mette in Input Mode.

### **Navigazione Rapida (Command Mode)**

* gg: Vai alla **prima riga** del file.  
* G: Vai all'**ultima riga** del file (es. 5G va alla riga 5).  
* ^: Vai all'**inizio** della riga corrente.  
* $: Vai alla **fine** della riga corrente.

### **Operazioni Rapide (in Command Mode)**

* x: Cancella un carattere sotto il cursore.  
* r: Sostituisce un singolo carattere sotto il cursore senza entrare in Input Mode.  
* dd: Taglia l'intera riga. 10dd: Taglia 10 righe.  
* yy: Copia l'intera riga. 10yy: Copia 10 righe.  
* p: Incolla dopo il cursore. P: Incolla prima.  
* u: Undo (annulla ultima operazione).  
* .: Ripete l'ultimo comando impartito.

### **Direttive e Ricerca (: e /)**

* :w: Salva.  
* :q\!: Esce ignorando i cambiamenti (fondamentale in caso di panico).  
* :wq o ZZ: Salva ed esce.  
* /pattern: Cerca "pattern" in avanti (n per il prossimo, N per il precedente).  
* **Sostituzione Globale letale:** :%s/testo\_vecchio/testo\_nuovo/g (Sostituisce tutto nel file. Aggiungi c alla fine per chiedere conferma riga per riga).

### **Macro (Operazioni Avanzate)**

* qa: Inizia a registrare una macro e la salva nella lettera a. (Premi di nuovo q per fermare la registrazione).  
* @a: Esegue la macro a. (10@a la ripete 10 volte).

*Hack the Planet. Ma prima, impara a usare bene la shell.*