# Laboratorio di Sicurezza Informatica — Sicurezza delle Applicazioni Web

**Docente:** Prof. Marco Prandini
**Dipartimento:** Informatica – Scienza e Ingegneria, Alma Mater Studiorum – Università di Bologna
**Argomento:** Architettura delle applicazioni web, tassonomia delle vulnerabilità web, OWASP Top Ten 2021.

---

## 1. Scenario: Architettura delle Applicazioni Web

Un'applicazione web moderna è articolata in più livelli, ognuno dei quali espone potenziali superfici di attacco:

| Livello | Componente tipico | Ruolo |
|---------|-------------------|-------|
| **Presentazione** | Browser (Chrome, Firefox, …) | Interfaccia utente; comunica via HTTP/HTTPS |
| **Web Server** | Apache, IIS, Nginx | Gestisce le richieste HTTP; punto di demarcazione rete |
| **Application Server** | Java, PHP, Rails, Node.js, Python | Contiene la *business logic* |
| **Data Access Layer** | MySQL, PostgreSQL, MongoDB | Persistenza dei dati |

La comunicazione tra browser e web server avviene mediante il protocollo HTTP. Ogni livello introduce **vulnerabilità specifiche**: la comprensione di questa stratificazione è il presupposto per una corretta analisi della sicurezza.

---

## 2. Tassonomia delle Vulnerabilità Web

Le vulnerabilità delle applicazioni web si classificano in tre macro-categorie:

### 2.1 Client Side

- **Errori di definizione del perimetro di interazione con i server:** il browser esegue operazioni non previste o non consentite.
- **Esecuzione di codice non fidato sul client:** es. Cross-Site Scripting (XSS).
- **Manipolazioni del DOM per ingannare l'utente:** es. Clickjacking, form spoofing.

### 2.2 Protocollo (HTTP)

- Il protocollo HTTP in sé non presenta vulnerabilità intrinseche; la **sicurezza del canale** è demandata a TLS (vedi A5).
- I problemi emergono da un **trattamento errato di cookie e dati serializzati** a livello di endpoint (server e/o client).
- **HTTP Request Smuggling:** ambiguità nell'interpretazione delle richieste HTTP tra server front-end e back-end in cascata, che può portare a bypass di controlli di sicurezza.

### 2.3 Server Side

- **Errori di controllo dell'accesso:** IDOR, File Disclosure, CSRF.
- **Errori di interpretazione dei dati:** XSS, SSRF, XXE, SQL/Command Injection, Deserialization.
- **Carenza di aggiornamento ed errata configurazione software.**

---

## 3. OWASP Top Ten

**OWASP** (*Open Web Application Security Project*, https://owasp.org/) è una fondazione no-profit che produce standard, strumenti e documentazione per migliorare la sicurezza del software. Il suo documento più noto è l'**OWASP Top Ten** (https://owasp.org/www-project-top-ten/), una lista aggiornata periodicamente dei dieci rischi più critici per la sicurezza delle applicazioni web, frutto di ampio consenso nella comunità.

L'**ultima versione ufficiale è quella del 2021** (https://owasp.org/Top10/). Rispetto alla lista del 2017 introduce tre nuove categorie e riorganizza significativamente le precedenti.

| Posizione 2021 | Categoria | Tendenza rispetto al 2017 |
|----------------|-----------|--------------------------|
| A01 | Broken Access Control | ↑ da A5 |
| A02 | Cryptographic Failures | ↑ da A3 |
| A03 | Injection | ↓ da A1 (ingloba XSS) |
| A04 | Insecure Design | **Nuova** |
| A05 | Security Misconfiguration | ↑ da A6 (ingloba XXE) |
| A06 | Vulnerable and Outdated Components | ↑ da A9 |
| A07 | Identification and Authentication Failures | ↓ da A2 |
| A08 | Software and Data Integrity Failures | **Nuova** (ingloba Insecure Deserialization) |
| A09 | Security Logging and Monitoring Failures | ↑ da A10 |
| A10 | Server-Side Request Forgery (SSRF) | **Nuova** |

---

## 4. A01 – Broken Access Control

### 4.1 Descrizione Generale

Raggruppa varie cause di **accesso non correttamente mediato alle risorse**. È salita al primo posto della classifica perché estremamente diffusa. Il meccanismo alla base è spesso la cosiddetta *security through obscurity*: l'applicazione non protegge esplicitamente le risorse, ma si limita a non pubblicarne i riferimenti, affidando la sicurezza alla sola difficoltà di indovinarli. Questa strategia è intrinsecamente fragile.

### 4.2 IDOR – Insecure Direct Object Reference

Una risorsa viene erogata semplicemente **perché se ne conosce l'identificatore**, senza verificare che il richiedente abbia il diritto di accedervi.

**Esempio pratico:**
```
GET /userapp.php → accesso come utente normale
```
L'attaccante modifica manualmente l'URL:
```
GET /adminapp.php → accesso alle funzionalità di amministrazione!
```
Oppure, con parametri espliciti:
```
https://example.com/fileop.php?f=a.txt&action=backup
https://example.com/fileop.php?f=a.txt&action=delete  ← azione privilegiata
```

**Mitigazioni:**
- Non esporre mai dati o identificatori direttamente dal server web.
- Usare **mappature effimere** con ID non prevedibili (es. hash crittografici) al posto di identificatori sequenziali.
- Applicare la verifica **AAA** (Authentication, Authorization, Accounting) a ogni singola richiesta.

### 4.3 FD – File Disclosure

È un caso particolare di IDOR in cui l'oggetto referenziato è un **elemento del filesystem**. Il caso classico è il **path traversal**: sfruttando la sequenza `../`, un attaccante risale l'albero delle directory oltre la radice prevista dall'applicazione.

**Esempio:**
```php
// Il server accetta il parametro 'doc' e blocca i caratteri shell (; & && ||)
<?php shell_exec("cat ".$VerifiedHome."/".$_GET["doc"]) ?>

// Ma se lascia liberi i caratteri validi per i path:
doc = ../../../../etc/passwd
// Il comando eseguito diventa:
cat /home/maybe/deep/username/../../../../etc/passwd
// Equivalente a: cat /etc/passwd
```

---

## 5. A02 – Cryptographic Failures

### 5.1 Descrizione

Precedentemente denominata *Sensitive Data Exposure*, questa categoria è stata rinominata per sottolineare che il problema originario è quasi sempre un **fallimento crittografico** sottostante. Le applicazioni web trattano grandi quantità di dati sensibili: PII (Personally Identifiable Information), dati sanitari, finanziari e credenziali di accesso.

Tali dati devono essere protetti sia **at rest** (a riposo, in database e storage) sia **in transit** (in transito sulla rete). I problemi emergono quando:

- Un dato non viene riconosciuto come sensibile.
- Non si individuano e proteggono tutte le copie del dato (es. log, backup, file temporanei).
- Non si coprono tutte le fasi del ciclo di vita del dato sensibile.
- Si utilizzano metodi crittografici inadeguati o obsoleti.

### 5.2 Esempio Notevole

Un sistema può cifrare correttamente il canale (TLS) e il database (cifratura trasparente), ma se il dato viene **copiato incidentalmente** in un file di log non protetto, la protezione viene di fatto aggirata.

**Scenario tipico:**
1. L'utente inserisce il numero di carta di credito in un form; il canale è cifrato con TLS.
2. Un errore del gateway di pagamento causa il salvataggio del numero nei log di debug.
3. I log sono accessibili a tutto il personale IT per scopi di troubleshooting.
4. Un insider malintenzionato esfilttra milioni di numeri di carta.

La protezione *at rest* del database era stata automaticamente rimossa prima che il dato venisse scritto nel log non protetto.

---

## 6. A03 – Injection

### 6.1 Principio Generale

Il principio alla base di tutte le forme di injection è l'**invio di input non fidati a un interprete**. Se il server non separa correttamente i dati dai comandi, l'input dell'utente può essere interpretato come codice, portando a violazioni dei principi CIA (Confidentiality, Integrity, Availability).

**Scenario generico:**
Il server costruisce un comando per un'applicazione esterna concatenando parti statiche con i parametri ricevuti via HTTP:
```php
<?php shell_exec("ls -l /home/" . $_GET["user"]) ?>
```
Se l'utente fornisce:
```
user = ; cat /etc/passwd
```
Il comando eseguito diventa:
```bash
ls -l /home/; cat /etc/passwd
```

### 6.2 SQL Injection

L'interprete bersaglio è il **DBMS**. Il server costruisce una query SQL incorporando i parametri dell'utente:

```sql
SELECT * FROM Users WHERE uid='$username' AND pwd='$password';
```

Inserendo come password `' OR 'a'='a`, la query diventa:
```sql
SELECT * FROM Users WHERE uid=* AND pwd='' OR 'a'='a';
```
Questa condizione è sempre vera: l'attaccante ottiene accesso senza conoscere la password.

Inserendo come password `'; DROP DATABASE WebApp; --`, la query diventa:
```sql
SELECT * FROM Users WHERE uid='' AND pwd=''; DROP DATABASE WebApp; --';
```
Con conseguente cancellazione dell'intero database.

**Mitigazioni:**
- **Bind variables** (*prepared statements*): separare struttura della query dai dati, impedendo che i parametri vengano interpretati come SQL.
- **Principio del minimo privilegio:** l'account DB usato dall'applicazione non deve avere permessi DDL.
- Sanitizzazione dell'input come difesa in profondità (non sufficiente da sola).

### 6.3 Cross-Site Scripting (XSS)

XSS è una forma di injection di codice **lato browser**: il codice malevolo (JavaScript) viene iniettato in una pagina web servita da un'applicazione vulnerabile e viene eseguito nel browser della vittima che si fida di quel sito. Tra il 2013 e il 2017 era una categoria OWASP autonoma (A7); nel 2021 è stata ricondotta all'interno di A03 per la sua natura di injection.

Le **tre modalità di esecuzione** sono:

#### 6.3.1 Reflected XSS

Il codice malevolo è inserito direttamente nell'URL e viene "riflesso" dal server nella pagina di risposta. Il server è vulnerabile perché non sanifica i parametri prima di includerli nell'output HTML.

**Esempio:**
```
URL legittimo: welcomepage.php?name=John
```
Il server produce:
```php
<?php echo 'Welcome to our site ' . stripslashes($_GET['name']); ?>
```
L'attaccante costruisce l'URL:
```
welcomepage.php?name=<script language=javascript>alert('Hijacked!');</script>
```
Il browser della vittima riceve ed esegue lo script.

**Flusso d'attacco:**
1. L'attaccante identifica la vulnerabilità XSS nell'applicazione.
2. Costruisce il payload JavaScript opportuno.
3. Lo invia alla vittima tramite social engineering (email, IM, social network).
4. La vittima (già autenticata) interagisce con il link; il codice viene eseguito nel suo browser.
5. L'attaccante raccoglie le credenziali o il token di sessione esfiltrato.

#### 6.3.2 DOM XSS

Simile al reflected, ma il codice vulnerabile che processa i dati in modo insicuro non è sul server, **è già nella pagina stessa** (JavaScript client-side). Il DOM ha accesso a una grande quantità di informazioni dell'utente.

**Esempio:**
```javascript
// Script nella pagina:
document.write("Site is at: " + document.location.href + ".");
// L'utente viene indotto a visitare:
http://www.yourdomain.com/brokenpage.html#<script>alert('Hijacked!');</script>
```

#### 6.3.3 Stored XSS (Persistent XSS)

Il payload non viene riflesso immediatamente: viene **salvato nel database** del server (es. tramite un form di commento o un messaggio). Quando qualsiasi utente successivo carica la pagina, il payload viene estratto dal DB e inviato al browser, dove viene eseguito. È la forma più pericolosa perché non richiede che la vittima clicchi su un link preparato ad hoc.

**Effetti possibili di XSS:**
- Dirottamento di sessioni autenticate (furto di cookie).
- Presentazione di form contraffatti per sottrarre credenziali o elementi MFA.
- Scaricamento di malware.
- Key logging silenzioso.

### 6.4 Injection: Approcci al Pentesting

| Approccio | Descrizione | Uso |
|-----------|-------------|-----|
| **White Box** | Si dispone del codice sorgente; si analizza staticamente | Revisione del codice |
| **Black Box** | Si stimola l'applicazione con input ragionati; si osserva l'output | Test esterno |
| **Blind Injection** | L'output non rivela dati ma solo successo/errore | DB senza output diretto |

Per estrarre informazioni in modalità **blind**:
- Si inietta un'operazione lenta (es. `SLEEP(5)`) e si misura il tempo di risposta.
- Si osservano le differenze nelle pagine di errore.
- Si inietta un'operazione di *pingback* verso un server controllato dall'attaccante.

---

## 7. A04 – Insecure Design

Categoria **nuova nel 2021**: attira l'attenzione sulla necessità di integrare la sicurezza già nella **fase di progettazione**, non solo nella fase di testing. Il difetto non è un bug nell'implementazione, ma nell'architettura stessa.

Metodi di progetto formalizzati raccomandati:
- **Threat Modeling:** identificazione sistematica delle minacce durante la fase di design.
- **Secure Design Patterns:** uso di pattern architetturali che incorporano garanzie di sicurezza.
- **Reference Architectures:** architetture di riferimento validate per specifici domini.

Il paradigma chiave è lo **Shift Left**: anticipare le attività di testing e sicurezza il più possibile a sinistra nel ciclo di sviluppo software (SDLC), già dalla scrittura del nuovo codice, invece di concentrarle solo nelle fasi finali prima del deployment. Questo riduce drasticamente il costo di correzione dei difetti.

---

## 8. A05 – Security Misconfiguration

### 8.1 Descrizione

Categoria molto ampia che raccoglie **errori di configurazione a tutti i livelli dello stack**: servizi di rete, web server, application server, database, framework, codice custom, VM, container, storage. Gli errori tipici includono:

- Mancata applicazione del **principio del minimo privilegio**.
- Funzioni non necessarie installate o abilitate.
- **Credenziali di default** non modificate (es. admin/admin su WordPress, database accessibili con utenti di default).
- Diagnostica abilitata in produzione che esfilttra dettagli interni.
- Regressione a default insicuri dopo aggiornamenti.
- Scelta di modalità di funzionamento insicure.

### 8.2 Security Misconfiguration a Livello di Protocollo: TLS

La scelta di cifrari deboli per TLS costituisce una misconfiguration critica.

**Cifrari e versioni da evitare:**
- SSL, TLSv1.0, TLSv1.1 (versioni obsolete e vulnerabili)
- RC2, RC4, Null, Export (cifrari deboli)
- DES, SHA-1

**Standard minimi raccomandati:**
- **TLSv1.2** (o superiore)
- Cifrari a **128+ bit**
- Cifrari con proprietà di **Forward Secrecy** (PFS)
- Chiavi asimmetriche con modulo **2048+ bit**
- **SHA-2** per le funzioni hash

Strumenti per la valutazione della configurazione HTTPS:
- https://www.ssllabs.com/ssltest/analyze.html
- https://www.htbridge.com/ssl/

### 8.3 Security Misconfiguration a Livello di Client: HTTP Security Headers

Il server può istruire il browser mediante **HTTP response header** per mitigare diverse classi di attacchi. La loro mancata configurazione costituisce una misconfiguration lato client.

#### Controllo del Contenuto della Pagina

| Header | Funzione |
|--------|----------|
| `X-Frame-Options: DENY \| SAMEORIGIN \| ALLOW-FROM <url>` | Impedisce l'embedding in `<frame>`, `<iframe>`, `<object>` (difesa anti-Clickjacking) |
| `X-XSS-Protection: 1; mode=block` | Filtra i tentativi di Reflected XSS (legacy; sostituito da CSP) |
| `Content-Security-Policy: default-src 'self'` | Previene il caricamento di risorse (es. JS) da origini esterne non autorizzate |

#### Gestione dei Dati HTTP

| Header | Funzione |
|--------|----------|
| `X-Content-Type-Options: nosniff` | Previene il MIME-sniffing; il browser usa solo il Content-Type dichiarato |
| `Cache-Control: no-store` | Impedisce il caching di risposte contenenti dati sensibili |
| `Pragma: no-cache` | Compatibilità HTTP 1.0 |
| `Expires: 0` | Compatibilità HTTP 1.0 |

#### Autenticità dei Siti

| Header | Funzione |
|--------|----------|
| `Strict-Transport-Security: max-age=<time>` (**HSTS**) | Impone al browser di interagire col server esclusivamente via HTTPS; previene attacchi di HTTP stripping e splitting |
| `Public-Key-Pins: pin-sha256=...` | Lega una chiave pubblica specifica a un dominio, contro la falsificazione di certificati X.509 |

Strumento per verificare la corretta configurazione degli header: https://securityheaders.io/

### 8.4 Same Origin Policy (SOP) e CORS

#### Same Origin Policy

Per isolare le risorse di siti distinti, il browser implementa la **Same Origin Policy (SOP)**: pagine e script caricati da un'origine possono accedere unicamente a risorse della stessa origine. Due risorse condividono la stessa origine se hanno lo stesso schema, host e porta. La SOP impedisce, ad esempio, che uno script iniettato dal dominio A legga i cookie settati dall'interazione col dominio B.

#### Cross-Origin Resource Sharing (CORS)

La SOP è necessaria ma troppo restrittiva per le applicazioni web moderne, che spesso devono accedere a risorse di domini diversi (CDN, API, font). **CORS** è il meccanismo standardizzato per rilassare i vincoli SOP in modo controllato e granulare.

**Funzionamento base:**
1. Il browser aggiunge l'header `Origin: domain-a.com` alle richieste cross-origin.
2. Il server `domain-b.com` risponde con `Access-Control-Allow-Origin` dichiarando le origini autorizzate.
3. Il browser verifica che l'origine del documento sia inclusa nell'elenco.

**Header CORS principali:**

| Tipo | Header |
|------|--------|
| **Request** | `Origin`, `Access-Control-Request-Method`, `Access-Control-Request-Headers` |
| **Response** | `Access-Control-Allow-Origin`, `Access-Control-Allow-Credentials`, `Access-Control-Allow-Methods`, `Access-Control-Allow-Headers`, `Access-Control-Expose-Headers`, `Access-Control-Max-Age` |

> **Attenzione:** CORS è un meccanismo lato browser. Una configurazione permissiva lato server (es. `Access-Control-Allow-Origin: *` con credenziali) non va compensata abbassando la guardia lato server. Riferimento: https://portswigger.net/web-security/cors

### 8.5 XML External Entities (XXE) — ora inclusa in A05

Era una categoria autonoma in OWASP 2017 (A04). XML permette di referenziare **entità esterne** tramite il DTD (*Document Type Definition*). Se un parser XML non è configurato correttamente, un documento XML malevolo può abusare di questo meccanismo per:

- **Leggere file dal server** (File Disclosure via path traversal).
- Effettuare attacchi **SSRF** (Server-Side Request Forgery).
- Inviare dati a sistemi controllati dall'attaccante.
- Attacchi **blind** che esfiltrano dati dai messaggi di errore.
- Attacchi **DoS** tramite espansione esponenziale di entità (*Billion Laughs Attack*).

**Esempio di lettura file tramite XXE:**
```xml
<!-- Richiesta legittima -->
<?xml version="1.0" encoding="UTF-8"?>
<stockCheck><productId>381</productId></stockCheck>

<!-- XML malevolo -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<stockCheck><productId>&xxe;</productId></stockCheck>

<!-- Risposta del server vulnerabile -->
Invalid product ID: root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin ...
```

**Mitigazioni:**
- Se possibile, usare formati più semplici come **JSON** invece di XML.
- Usare parser XML aggiornati.
- **Disabilitare il supporto alle external entity** (almeno per i DTD).
- Sanitizzare l'input consentendo solo gli elementi XML necessari.

---

## 9. A06 – Vulnerable and Outdated Components

Categoria autoesplicativa ma di grande impatto pratico. L'uso di componenti software (librerie, framework, OS, database, VM) con vulnerabilità note espone l'intera applicazione ai rischi associati a quelle vulnerabilità, indipendentemente dalla qualità del codice applicativo.

**Esempi storici notori:**
- **Heartbleed** (CVE-2014-0160): OpenSSL — lettura arbitraria di memoria del processo server.
- **ShellShock** (CVE-2014-6271): Bash — esecuzione arbitraria di codice via variabili d'ambiente.
- **GHOST** (CVE-2015-0235): glibc Linux — buffer overflow nel resolver DNS.
- **DROWN** (CVE-2016-0800): OpenSSL — decifrazione di sessioni TLS tramite SSLv2.

Il numero di CVE pubblicate cresce ogni anno. Mantenere aggiornati tutti i componenti di uno stack moderno è **tecnicamente difficile ma imprescindibile**. Strumenti di monitoraggio: https://vulnerability-watch.connettiva.eu/

L'impatto potenziale è qualsiasi: Remote Code Execution (RCE), violazioni AAA, esfiltrazione di dati, DoS.

---

## 10. A07 – Identification and Authentication Failures

### 10.1 Descrizione

Precedentemente denominata *Broken Authentication* (A02 nel 2017). Le applicazioni web fanno largo uso di **sessioni** per mantenere lo stato autenticato:

1. L'utente si autentica con le proprie credenziali.
2. Il server assegna un **token segreto** che identifica la sessione.
3. Il browser include il token in ogni richiesta successiva (tipicamente via cookie).
4. Il server riconosce la sessione come autenticata verificando il token.

La gestione non corretta dell'autenticazione e del token di sessione può consentire a un attaccante di **impersonare un utente legittimo**.

### 10.2 Session Fixation

L'attaccante avvia una sessione con il server (ottenendo un `sessionID`), poi convince la vittima a proseguire la propria navigazione usando quel `sessionID` (es. tramite un link con `?sessionID=123XYZ`). Quando la vittima si autentica, il server associa l'autenticazione a quel `sessionID` che l'attaccante già conosce.

**Condizioni di vulnerabilità dei token di sessione:**
- L'ID è **prevedibile** (es. sequenziale).
- L'ID è **intercettabile** (trasmesso in chiaro anziché via HTTPS).
- L'ID non è legato strettamente all'utente e alla postazione (può essere copiato e riusato).
- L'ID non scade dopo un periodo di inattività.
- L'ID non viene **invalidato** al termine della sessione (logout).

### 10.3 Cross-Site Request Forgery (CSRF)

Il browser include **automaticamente** tutti i cookie di sessione in ogni richiesta inviata al dominio corrispondente, indipendentemente dall'origine della pagina che ha generato la richiesta. CSRF sfrutta questo meccanismo.

**Flusso d'attacco:**
1. La vittima si autentica su `mybank.com` (il suo browser memorizza il cookie di sessione).
2. La vittima visita `attacker.com`, che contiene una pagina con:
```html
<form action="https://mybank.com/transfer.jsp" method="POST">
  <input name="recipient" value="attacker">
  <input name="amount" value="1000">
</form>
<script>document.forms[0].submit()</script>
```
3. Il browser invia automaticamente la richiesta POST a `mybank.com` **includendo il cookie di sessione**.
4. Il server di `mybank.com` non può distinguere questa richiesta da una legittima: il bonifico viene eseguito.

**Mitigazioni per CSRF:**
- Il server legittimo include un **segreto anti-CSRF** in ogni form: univoco, non falsificabile, non intercettabile via URL, es. `<input type="hidden" value="23a3af01b>`.
- Il server valida il campo `Referer` o richiede l'uso di **HTTP header custom** (che le richieste cross-origin non possono includere senza CORS esplicito).

---

## 11. A08 – Software and Data Integrity Failures

### 11.1 Descrizione

Categoria **nuova nel 2021** (incorpora la precedente A08 *Insecure Deserialization*). Riguarda le vulnerabilità risultanti da un'**insufficiente tutela dell'integrità** di dati, codice e infrastruttura:

- Uso di plugin da fonti esterne non verificate.
- Uso di pipeline CI/CD automatizzate che consentono di mascherare iniezioni di codice malevolo.
- Livelli di autenticazione ridotti durante le fasi di aggiornamento rispetto all'installazione.

**Esempio di risonanza mondiale:** attacco **SolarWinds Orion** (2020) — malware iniettato nel processo di build automatizzato ha infettato circa 18.000 organizzazioni, tra cui Microsoft, Intel e Cisco.

### 11.2 Insecure Deserialization

La **serializzazione** è il processo di conversione di un oggetto strutturato in un flusso sequenziale di byte, adatto alla trasmissione in rete o alla memorizzazione. La **deserializzazione** è il processo inverso. Sono usate estensivamente nelle applicazioni web per: web services, message broker, storage (cache, DB), cookie, token di autenticazione nelle API.

**Condizioni di vulnerabilità:**
- L'oggetto serializzato viene passato **in chiaro via HTTP** o memorizzato senza adeguato controllo degli accessi.
- Il deserializzatore non verifica l'**integrità** dello stream ricevuto prima di ricostruire l'oggetto.

**Esempi di effetti possibili:**
- Passaggio di oggetti Java o .NET manipolati → **esecuzione di codice arbitrario (RCE)**.
- Passaggio di cookie con ruolo dell'utente autenticato modificato → **privilege escalation**.

**Mitigazioni:**
- Non deserializzare dati non fidati, ove possibile.
- Usare solo librerie con **controllo stretto dei tipi**; non lasciare mai che il tipo dell'oggetto venga determinato analizzando lo stream stesso.
- Monitoraggio (rilevazione di tentativi di brute force).
- Sandboxing del processo di deserializzazione.

---

## 12. A09 – Security Logging and Monitoring Failures

Non è una vulnerabilità in senso stretto, ma un **errore procedurale** che facilita il lavoro dell'attaccante: senza log adeguati, un attacco può passare inosservato per settimane o mesi.

**Errori comuni:**
- Non tracciare accessi falliti, transazioni terminate con errori, invocazioni di API non corrette.
- Non dettagliare sufficientemente gli eventi loggati.
- Non proteggere adeguatamente i log (integrità, riservatezza, conservazione per tempo adeguato).
- Non definire o non seguire procedure di risposta agli incidenti.

**Effetti comuni:**
- Non si rilevano tentativi di forza bruta.
- Non si riesce a ricostruire la dinamica di un attacco per mitigare la vulnerabilità utilizzata.
- Non si riesce a riparare il danno subito, in parte o del tutto.

---

## 13. A10 – Server-Side Request Forgery (SSRF)

Categoria **nuova nel 2021**. Si verifica quando un'applicazione web lato server:
1. Riceve dall'utente una **URL** da cui recuperare una risorsa remota.
2. Non esegue verifiche adeguate sull'URL fornita.
3. La utilizza per effettuare una richiesta HTTP verso quella risorsa.

Poiché la richiesta viene effettuata **dal server stesso**, l'attaccante può indirizzarla verso risorse interne normalmente non accessibili dall'esterno.

**Risultati possibili:**
- **Aggiramento di firewall** o altre misure di controllo dell'accesso perimetrale.
- **Enumerazione ed esfiltrazione di risorse interne** (es. metadati di istanze cloud, servizi su `localhost`).
- **Esecuzione di codice remoto** su sistemi non direttamente accessibili dall'esterno.

---

## Riepilogo dei Concetti Chiave

| Concetto | Definizione Sintetica |
|----------|-----------------------|
| **OWASP Top Ten** | Lista dei 10 rischi più critici per la sicurezza delle web app, aggiornata al 2021 |
| **IDOR** | Accesso a una risorsa tramite conoscenza del suo identificatore, senza verifica autorizzativa |
| **Path Traversal** | Uso di `../` per accedere a file al di fuori della directory prevista |
| **SQL Injection** | Input malevolo interpretato come SQL dal DBMS |
| **Reflected XSS** | Payload JS nell'URL, riflesso immediatamente nella risposta HTML |
| **Stored XSS** | Payload JS persistito nel DB e servito a tutti gli utenti successivi |
| **DOM XSS** | Payload JS processato da codice client-side vulnerabile nel DOM |
| **Blind Injection** | Injection senza output diretto; si inferisce il risultato dal comportamento del sistema |
| **CSRF** | Richiesta forgiata da un sito terzo, eseguita col token di sessione della vittima |
| **Session Fixation** | L'attaccante forza la vittima a usare un `sessionID` già noto |
| **XXE** | XML External Entities; abuso del parser XML per leggere file o effettuare SSRF |
| **CORS** | Meccanismo per rilassare la Same Origin Policy in modo controllato |
| **SOP** | Same Origin Policy; isolamento delle risorse tra origini diverse nel browser |
| **HSTS** | HTTP Strict Transport Security; impone HTTPS al browser |
| **Insecure Deserialization** | Oggetto serializzato manipolato per ottenere RCE o privilege escalation |
| **SSRF** | Il server viene indotto a fare richieste verso risorse interne non accessibili direttamente |
| **Shift Left** | Anticipare sicurezza e testing alle fasi iniziali del ciclo di sviluppo |
