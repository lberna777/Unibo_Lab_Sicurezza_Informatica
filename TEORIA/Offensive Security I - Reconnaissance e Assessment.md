# Laboratorio di Sicurezza Informatica — Offensive Security I: Reconnaissance & Assessment

**Docente:** Prof. Marco Prandini
**Dipartimento:** Informatica – Scienza e Ingegneria, Alma Mater Studiorum – Università di Bologna
**Argomento:** Ciclo di vita delle vulnerabilità, metodologie di Offensive Security, OSINT, enumerazione e Vulnerability Assessment.

---

## 1. Dove si Annidano le Vulnerabilità

Le vulnerabilità sono presenti a ogni livello dello stack tecnologico:

- **Livello Applicativo (App):** difetti di logica, input validation, injection, ecc.
- **Sistema Operativo (OS):** kernel, driver, servizi di sistema.
- **Hardware e interfacce I/O:** firmware, device driver, canali di accesso fisico.

Questa stratificazione implica che la superficie d'attacco di un sistema reale è la somma di tutti i possibili vettori presenti a ciascun livello.

---

## 2. Il Ciclo di Vita della Vulnerabilità

Una vulnerabilità attraversa diverse fasi dalla sua nascita alla sua mitigazione:

1. **Scoperta:** un ricercatore (o un attaccante) identifica il difetto.
2. **Sviluppo dell'exploit:** viene costruito un codice o una procedura per sfruttarla.
3. **Pubblicazione (responsible disclosure):** idealmente il ricercatore notifica il vendor prima della divulgazione pubblica, per permettere lo sviluppo di una patch.
4. **Rilascio della patch:** il vendor corregge il difetto.
5. **Applicazione della patch:** gli amministratori aggiornano i sistemi esposti.

### Vulnerabilità Zero-Day

Una **vulnerabilità zero-day** è una vulnerabilità *sconosciuta* a chi dovrebbe mitigarla (vendor, amministratori). La **finestra di opportunità** è il periodo che intercorre tra la comparsa del primo exploit attivo e il momento in cui la patch viene distribuita e applicata. Durante questa finestra il sistema è esposto senza possibilità di difesa tramite aggiornamento.

> **Esempio pratico:** CVE-2024-4367 — una vulnerabilità nel lettore PDF integrato in Firefox (scritto in JavaScript) che permetteva l'esecuzione arbitraria di codice JavaScript inviando un file PDF malevolo. Il ricercatore che la scoprì la divulgò responsabilmente e la patch fu rilasciata in tempi rapidi, dimostrando come un processo di responsible disclosure efficiente riduca drasticamente la finestra di opportunità.

### Pubblicazione delle Vulnerabilità

La comunità della sicurezza mantiene database pubblici di riferimento per la gestione delle vulnerabilità note:

- **CVE (Common Vulnerabilities and Exposures):** http://cve.mitre.org/ — identificatore univoco per ogni vulnerabilità.
- **NVD (National Vulnerability Database):** http://nvd.nist.gov/ — arricchisce i CVE con metriche di severità (CVSS).
- **OSVDB / OSV:** database open source.

---

## 3. Offensive Security: Definizione e Metodologia

### Cosa Significa "Offensive Security"

Fare *offensive security* significa **porsi nel ruolo dell'attaccante** per:

- verificare l'esistenza di vulnerabilità prima che lo faccia un avversario reale;
- stimare con precisione l'impatto degli attacchi;
- testare l'efficacia delle contromisure adottate.

Lo strumento concettuale fondamentale è la **Kill Chain**: un modello che descrive la sequenza di fasi che un attaccante attraversa per raggiungere il proprio obiettivo.

Il framework di riferimento operativo è **MITRE ATT&CK** (https://attack.mitre.org/), che cataloga tattiche, tecniche e procedure (TTP) degli avversari.

> **Nota etica e legale:** usare le stesse tecniche degli attaccanti è un'attività delicata. È **assolutamente vietato** condurre qualsiasi test su risorse che non si possiede o per cui non si dispone di un'autorizzazione esplicita e formale. Il termine "permesso" va inteso in senso ampio: anche operazioni condotte in buona fede senza autorizzazione possono avere conseguenze legali e causare danni imprevisti ai sistemi bersaglio.

### Reconnaissance come Primo Anello della Kill Chain

La fase di *Reconnaissance* (https://attack.mitre.org/tactics/TA0043/) è il punto di partenza di qualsiasi attività offensiva. In questo corso essa comprende:

- **Enumerazione** delle risorse esposte.
- **Scansione** di host e servizi.
- **Brute forcing** di credenziali o configurazioni.
- **Vulnerability Assessment** dell'esposizione dei sistemi.

---

## 4. Vulnerability Assessment (VA) vs Penetration Testing (PT)

### VA vs PT: Differenze Fondamentali

Il confine tra le due metodologie è netto e spesso incompreso:

### Vulnerability Assessment (VA)

Il VA identifica e classifica le vulnerabilità note presenti in un sistema, senza procedere oltre: non tenta di sfruttarle. Usa strumenti automatici che confrontano le versioni dei servizi rilevati con i database di vulnerabilità (CVE/NVD). Può produrre **falsi positivi** (es. un servizio che dichiara una versione vulnerabile ma è in realtà già patchato).

### Penetration Testing

Il PT va oltre il VA: **sfrutta** le vulnerabilità trovate per ottenere un accesso più interno e approfondito al sistema, rivelando vulnerabilità che il VA non potrebbe individuare in autonomia. È un processo che considera la specificità del sistema target e simula un attaccante reale.

### Fasi del Penetration Test

1. **Valutazione del target:** definizione delle regole di ingaggio, mappatura e prioritizzazione del perimetro.
2. **Postura e visibilità:** scelta tra black-box (attacco cieco, più realistico ma più lento) e white-box/grey-box (l'esperto conosce parzialmente il sistema, più efficiente).
3. **Scelta della metodologia:** esistono standard generalmente accettati, tra cui:
   - *OSSTMM* (Open Source Security Testing Methodology Manual)
   - *PTES* (Penetration Testing Execution Standard)
   - *OWASP* (per le applicazioni web)
   - *NIST SP 800-115*

---

## 5. OSINT — Open Source Intelligence

### Definizione

**OSINT** (*Open Source INTelligence*) è l'uso di qualsiasi fonte *pubblicamente disponibile* per ricavare informazioni su un obiettivo specifico. È un campo più ampio della sola cybersecurity: viene utilizzato anche in intelligence, giornalismo investigativo e analisi geopolitica.

In ambito offensivo, l'OSINT costituisce la **fase di Reconnaissance** e permette di ampliare il perimetro di test raccogliendo informazioni prima di toccare direttamente i sistemi bersaglio (approccio *passivo*).

### Strumenti e Fonti Online

Il framework di riferimento per orientarsi nell'ecosistema OSINT è: https://osintframework.com/

Per misurare l'esposizione infrastrutturale di un target le tecniche principali includono:

- **Collocazione fisica e geolocalizzazione** degli asset.
- **Rilevazione di indirizzi IP** da documenti, pagine web e DNS.
- **Identificazione di domini** e sottodomini DNS associati all'obiettivo.
- **Analisi WHOIS** per ottenere informazioni su registranti e blocchi IP allocati.

---

## 6. Google Dorking

### Cos'è il Dorking

Le **Google Dorks** (o *operatori di ricerca avanzata*) sono sequenze di parole chiave che affinano i risultati del motore di ricerca, consentendo di trovare risorse normalmente non visibili in una ricerca ordinaria. Rappresentano uno strumento OSINT potente perché sfruttano ciò che Google ha già indicizzato pubblicamente.

### Keyword Principali

| Operatore | Funzione |
|-----------|----------|
| `cache:domain` | Mostra la versione in cache di un sito |
| `allintext:keyword` | Cerca testo specifico all'interno delle pagine web |
| `allinurl:keyword` | Cerca pagine il cui URL contiene tutte le parole specificate |
| `filetype:ext` | Filtra per tipo di file (es. `filetype:pdf`, `filetype:log`) |
| `site:domain` | Limita la ricerca a un dominio specifico |
| `intitle:keyword` | Cerca pagine con quella parola nel titolo |
| `inurl:keyword` | Cerca pagine con quella parola nell'URL |
| `after:YYYY` | Filtra risultati dopo una data |

### Struttura di una Query

La struttura generale è: `"inurl:.<domain>/<dorks>"`

### Esempi Pratici

- **Credenziali nei log:** `allintext:password filetype:log after:2020`
- **Username nei log:** `allintext:username filetype:log`
- **Webcam live esposte:** `inurl:top.htm inurl:currenttime`
- **Server FTP aperti:** `intitle:"index of" inurl:ftp`
- **Chiavi SSH indicizzate:** `site:ulisse.unibo.it intitle:index.of id_rsa -id_rsa.pub`
- **File PDF con "password" su un sito specifico:** `site:ulisse.unibo.it filetype:PDF intext:password`

### Contromisura

Tutto ciò che Google trova è stato preventivamente indicizzato. Il file **`robots.txt`** permette al webmaster di istruire i crawler sui percorsi da non indicizzare:

```
User-agent: *
Disallow: /cartella_da_non_indicizzare/
```

---

## 7. Enumerazione dell'Infrastruttura di Rete

### Blocchi IP e IANA/RIR

Gli indirizzi IP pubblici sono assegnati dallo **IANA** ai **RIR** (Regional Internet Registries), che li sub-allocano ai **LIR** (Local Internet Registries). Partendo da un singolo indirizzo IP noto (es. estratto da un sito web dell'obiettivo), è possibile risalire all'intero blocco allocato all'organizzazione interrogando i database WHOIS dei RIR.

### DNS Enumeration

I record DNS possono rivelare informazioni preziose sull'infrastruttura del target:

- Gli **IP registrati** dall'obiettivo.
- L'esistenza e la collocazione di **server applicativi specifici** (mail, web, ecc.).
- L'esistenza di **sottoreti non direttamente raggiungibili** dall'esterno.
- **Alias** per sistemi collocati fuori dal perimetro dichiarato.

#### Tipi di Record DNS

| Record | Descrizione |
|--------|-------------|
| `A` | Mappa nome → indirizzo IPv4 |
| `AAAA` | Mappa nome → indirizzo IPv6 |
| `MX` | Mail Exchanger (server di posta) |
| `NS` | Name Server autoritativo |
| `CNAME` | Alias verso un altro nome DNS |
| `TXT` | Record testuale (spesso usato per SPF, DKIM, verifica di proprietà) |
| `PTR` | Reverse lookup (IP → nome) |
| `SOA` | Start of Authority, informazioni sul dominio |

#### Strumenti per DNS Enumeration

- **`nslookup`** — tool nativo per interrogare server DNS (disponibile su Linux e Windows).
- **`dnsrecon`** — enumera automaticamente tutti i tipi di record per un dominio: `dnsrecon -d DOMAIN`
- **`dnsmap`** — brute-forcing di sottodomini: `dnsmap DOMAIN`
- **Online:** https://centralops.net/ (Domain Dossier), https://dnsdumpster.com/, https://whois.domaintools.com/, https://reverseip.domaintools.com/
- **Shodan (https://shodan.io/)** — motore di ricerca specializzato per dispositivi connessi a Internet. Indicizza banner e metadati di servizi esposti (HTTP, SSH, FTP, Telnet, ecc.) permettendo di trovare sistemi vulnerabili, dispositivi IoT mal configurati e versioni software obsolete senza dover effettuare scansioni attive. È uno strumento OSINT di primo ordine per la fase di Reconnaissance passiva.

---

## 8. Subdomain Enumeration

### Perché è Cruciale

L'enumerazione dei sottodomini è una delle fasi più critiche della Reconnaissance. Amplia enormemente la superficie d'attacco, permettendo di individuare:

- **Domini nascosti o "minori"** non pubblicizzati ufficialmente.
- Applicativi esposti su sottodomini non protetti allo stesso livello del dominio principale.
- Sistemi legacy dimenticati.

Un **FQDN** (*Fully Qualified Domain Name*) identifica univocamente un host specifico (es. `miopc.ulisse.com`). Un sottodominio può non avere un applicativo web sulle porte standard (80/443) e tuttavia essere attivo su altre porte.

### Strategie: Passiva vs Attiva

**Strategia Passiva:** interroga dataset di DNS già noti (es. SecurityTrails, Censys, VirusTotal) senza mai contattare direttamente il target. È stealth per definizione.

**Strategia Attiva:** effettua query DNS dirette verso i server autoritativi del target o tenta il **DNS Brute-Forcing** (prova sottomini generati da wordlist). È più completa ma lascia tracce.

#### Strumenti Notevoli

- **`amass`** (OWASP Foundation) — aggrega passivamente risultati da molteplici fonti: https://github.com/OWASP/Amass
- **`subfinder`**, **`assetfinder`** — altri tool di aggregazione passiva.

### Certificate Transparency (CT) Abuse

Ogni certificato SSL/TLS emesso da una **CA** (*Certification Authority*) pubblica viene registrato in un **CT Log** (RFC 6962). Questi log sono consultabili pubblicamente. Poiché ogni sottodominio per cui viene emesso un certificato appare in questi log, è possibile enumerare i sottodomini di un'organizzazione interrogando i CT Log *senza mai contattare direttamente* i server del target.

Strumenti e risorse:
- https://crt.sh/ — interfaccia web per interrogare i CT Log.
- https://censys.io/ — motore di ricerca per asset Internet indicizzati via CT.

> **Nota:** non esiste una difesa efficace contro questa tecnica, poiché i sottodomini certificati sono per natura esposti nei CT Log pubblici. L'unica contromisura è la consapevolezza della propria superficie d'attacco.

---

## 9. Host Discovery e Scansione dei Servizi

### Host Discovery

Una volta individuati i blocchi di indirizzi da analizzare, si procede alla scoperta degli host effettivamente attivi (*live hosts*).

- **Ping scan:** banale ma spesso bloccato dai firewall da Internet.
- **nmap -sn (Ping Scan su LAN):** su reti locali usa richieste ARP in broadcast, aggirando i firewall ICMP. Esempio: `nmap -sn 192.168.56.0/24`

### Servizi e Porte

Ogni servizio raggiungibile è in ascolto su una **porta TCP o UDP**. Le *well-known ports* (0–1023) sono assegnate dalla IANA (file di riferimento: `/etc/services`). Un servizio può tuttavia essere spostato su porte non standard per *security through obscurity*.

### Enumerazione dei Servizi con nmap

`nmap` è il tool più diffuso per la scansione di porte e servizi.

- **`nmap -sT <target>`** — TCP connect scan (scansione base sulle porte standard).
- **`nmap -sV <target> -p-`** — *Version Detection*: scansiona tutte le 65535 porte TCP e invia probe ai servizi per identificare applicazione e versione esatta dal banner di risposta.

> **Automazione:** i risultati di nmap possono essere passati a piattaforme di VA come **Greenbone Security Assistant (OpenVAS / GCE)**, che incrociano automaticamente le versioni rilevate con i database CVE per identificare vulnerabilità note.

---

## 10. Evasione e Postura Interna

### Evasione (Stealth Reconnaissance)

La fase di OSINT ed enumerazione ha due scopi spesso in tensione tra loro:

1. **Testare l'efficacia degli IDS** (Intrusion Detection System).
2. **Evitare di essere rilevati dagli IPS** (Intrusion Prevention System) per proseguire il test.

Per la Reconnaissance in modalità anonima si usano:
- **TOR** o reti proxy per anonimizzare l'origine delle richieste.
- **Account usa e getta** per accedere a servizi che richiedono registrazione.

### Postura Interna

Reconnaissance ed enumeration condotte dall'esterno simulano fedelmente la postura di un attaccante remoto. Tuttavia, se questa fase non identifica un vettore di intrusione, si può optare per un **auto-attacco dall'interno** (simulando un insider threat o un attaccante già all'interno del perimetro).

Modalità per guadagnare una postura interna:
- **Reti Wireless:** se ci si trova fisicamente a portata del segnale WiFi, è possibile tentare l'accesso alla rete e alle comunicazioni (tool: `aircrack-ng` — http://www.aircrack-ng.org/).
- **Attacchi WPA/WPA2:** cattura dell'handshake e brute forcing offline della passphrase.

---

## 11. Analisi dei Servizi Applicativi e Brute Forcing

Una volta identificati host e porte, si analizzano i protocolli applicativi esposti:

- **SMB, SMTP, SNMP** — protocolli spesso misconfigured che possono fornire accesso a dati o informazioni utili per le fasi successive.
- **Brute forcing applicativo (fuzzing)** — invio di payload randomizzati o da wordlist per tentare di accedere ai servizi (es. autenticazione SSH, HTTP, FTP).
- **Sfruttamento di misconfigurazioni** — accesso diretto a database esposti (MySQL, PostgreSQL), enumerazione di utenti SMTP via `VRFY`, ecc.

---

## 12. Strumenti di Vulnerability Assessment Completi

### Greenbone Community Edition (GCE) / OpenVAS

GCE è una piattaforma open source per il Vulnerability Assessment. È configurabile per eseguire la scansione di combinazioni arbitrarie di host e porte, e testa l'esistenza di vulnerabilità a livello di rete, sistema operativo e applicazione tramite plugin.

Il suo valore fondamentale risiede nel **Feed**: un database di **NVT** (*Network Vulnerability Tests*) aggiornato quotidianamente, dove ogni NVT contiene la descrizione della vulnerabilità, le piattaforme colpite e il processo di verifica automatica. Consulta: https://www.greenbone.net/en/feed-comparison/

> **Architettura di GCE:** si articola in uno scanner (OpenVAS Scanner), un gestore delle configurazioni (GVM — Greenbone Vulnerability Manager) e un'interfaccia web (GSA — Greenbone Security Assistant).

---

## Riepilogo dei Concetti Chiave

| Concetto | Definizione Sintetica |
|----------|-----------------------|
| **Zero-day** | Vulnerabilità sconosciuta a chi dovrebbe mitigarla |
| **Responsible Disclosure** | Notifica al vendor prima della pubblicazione pubblica |
| **Kill Chain** | Modello a fasi del processo di attacco |
| **MITRE ATT&CK** | Framework che cataloga TTP degli avversari reali |
| **VA** | Identifica vulnerabilità note senza sfruttarle |
| **PT** | Simula un attacco reale, sfruttando le vulnerabilità trovate |
| **OSINT** | Raccolta di informazioni da fonti pubblicamente disponibili |
| **Google Dorking** | Uso di operatori di ricerca avanzata per esporre risorse non protette |
| **DNS Enumeration** | Raccolta sistematica dei record DNS di un target |
| **Subdomain Enumeration** | Individuazione di sottodomini per ampliare la superficie d'attacco |
| **CT Log Abuse** | Enumerazione passiva dei sottodomini tramite i log Certificate Transparency |
| **nmap -sV** | Scansione con rilevamento della versione dei servizi |
| **GCE/OpenVAS** | Scanner di VA completo con feed di NVT aggiornato quotidianamente |
