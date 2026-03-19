# Laboratorio di Sicurezza Informatica — Firewall

**Docente:** Prof. Marco Prandini
**Dipartimento:** Informatica – Scienza e Ingegneria, Alma Mater Studiorum – Università di Bologna
**Argomento:** Difesa perimetrale, tipologie e topologie di firewall, architettura Netfilter/nftables in Linux, catene, tabelle e regole di packet filtering.

---

## 1. Firewall come Difesa Perimetrale

Il termine *firewall* — letteralmente "muro tagliafuoco" — indica un dispositivo (o un insieme di dispositivi) preposto a limitare la propagazione di traffico indesiderato tra zone di rete con diversi livelli di fiducia. La metafora architettonica più appropriata non è quella del muro, bensì quella di una **cinta muraria con una porta**: divide il "dentro" dal "fuori", e tutto il traffico è obbligato a transitare per quel singolo punto di controllo.

Questa centralizzazione produce due conseguenze fondamentali:

- **Politiche uniformi:** le regole di controllo dell'accesso sono definite e applicate in un unico punto, senza doverle replicare su ogni sistema della rete interna.
- **Funzionalità sofisticate in un punto unico:** è possibile implementare ispezione approfondita, logging, traduzione degli indirizzi e altri meccanismi avanzati senza gravare su ogni host della rete.

### Filtraggio in ingresso e in uscita

La porta della cinta muraria controlla il traffico in entrambe le direzioni:

- **Ingress filtering:** più intuitivo; impedisce l'accesso di soggetti non autorizzati alla rete interna.
- **Egress filtering:** altrettanto importante; impedisce l'esfiltrazione di dati riservati verso l'esterno e impedisce che i sistemi interni vengano utilizzati come base di lancio per attacchi verso terzi (ad esempio, come nodi di una botnet).

---

## 2. Principi Fondamentali

Un firewall efficace si fonda su quattro principi irrinunciabili:

**Punto di passaggio obbligato.** Il firewall è efficace soltanto se non esistono percorsi alternativi per raggiungere la rete protetta. Una connessione Wi-Fi non monitorata, un modem secondario o un dispositivo mobile mal configurato annullano le garanzie offerte dalla difesa perimetrale.

**Default deny.** Il principio cardine è opposto alla logica permissiva: è consentito *solo* ciò che è esplicitamente autorizzato da una regola. Tutto il traffico non previsto viene scartato per default. Questo approccio forza a ragionare in termini di *whitelist* di comportamenti leciti anziché di *blacklist* di comportamenti vietati.

**Robustezza.** Il firewall è un componente critico dell'infrastruttura di sicurezza e deve essere immune agli attacchi. È quindi tipicamente un sistema dedicato, in cui si rinuncia intenzionalmente a flessibilità e praticità in favore della riduzione della superficie di attacco. Un firewall compromesso è peggio dell'assenza di un firewall.

**Architettura, non prodotto.** Un firewall non è necessariamente un singolo dispositivo: è un'architettura composta da uno o più componenti hardware e/o software, la cui configurazione congiunta realizza la politica di sicurezza desiderata.

---

## 3. Tecniche di Controllo del Traffico

Un firewall può esaminare il traffico lungo quattro dimensioni:

**Traffico** (cosa): analisi di indirizzi, porte, protocolli e altri indicatori presenti negli header del pacchetto. È la tecnica più elementare e la più universalmente implementata.

**Direzione** (da dove verso dove): discriminazione tra richieste *entranti* verso la rete interna e richieste *uscenti* generate da essa. È importante notare che il traffico TCP/IP è sempre bidirezionale — ogni connessione produce pacchetti in entrambi i sensi — e che la *direzione logica* di una connessione è definita da chi ha preso l'iniziativa (chi ha inviato il primo pacchetto SYN), non dalla direzione dei singoli pacchetti.

**Utenti** (chi): differenziazione dell'accesso sulla base dell'identità del richiedente. Si tratta di una tecnica intrinsecamente limitata nei firewall di rete tradizionali, poiché il protocollo TCP/IP non trasporta negli header alcuna informazione sull'utente responsabile della generazione del pacchetto.

**Comportamento** (come): valutazione dell'uso dei servizi ammessi per identificare anomalie rispetto a parametri di normalità definiti a priori. È la tecnica più sofisticata, implementata nei sistemi di intrusion detection e nei firewall applicativi più avanzati.

---

## 4. Tipologie di Firewall

### 4.1 Packet Filter (PF)

Il packet filter è il tipo più elementare e diffuso. Esamina **esclusivamente gli header** del pacchetto — senza mai aprire il payload — e applica in sequenza un elenco ordinato di regole del tipo "se condizione → allora azione". La prima regola soddisfatta determina il destino del pacchetto e interrompe la scansione dell'elenco. Se nessuna regola viene attivata, si applica una **politica di default** (tipicamente: scartare il pacchetto, in coerenza con il principio *default deny*).

I campi esaminabili includono:

| Livello | Campi |
|---|---|
| **Link Layer** | Interfaccia fisica di ingresso/uscita, MAC address sorgente/destinazione |
| **IP Layer** | Indirizzi IP sorgente/destinazione, protocollo trasportato (ICMP, TCP, UDP, …), opzioni IP (TOS, ECN, …) |
| **Transport Layer** | Porte TCP/UDP sorgente/destinazione, flag TCP (SYN, ACK, FIN, RST, …) |

**Vantaggi:** semplicità, velocità di elaborazione (implementato in hardware su quasi tutti i router), trasparenza agli utenti finali (non richiede riconfigurazioni lato client).

**Svantaggi:** regole di basso livello (comportamenti sofisticati richiedono set di regole molto complessi); assenza nativa di supporto alla gestione degli utenti.

#### Vulnerabilità e limitazioni del PF

**Frammentazione IP.** I frammenti successivi al primo di un datagramma frammentato non contengono l'header di trasporto (TCP/UDP): le condizioni che verificano porte o flag TCP non possono essere valutate su di essi. Questo permette a un attaccante di eludere il filtraggio inviando traffico vietato in frammenti. La soluzione drastica è scartare tutti i pacchetti frammentati; la soluzione corretta (ma computazionalmente costosa) è riassemblarli prima dell'analisi, operazione non implementabile in un PF puro.

**IP Spoofing.** La falsificazione dell'indirizzo sorgente di un pacchetto non è rilevabile dal PF senza un controllo di coerenza tra indirizzo e interfaccia di arrivo. Le best practice (RFC 2827, RFC 3704, RFC 8704) raccomandano di verificare che i pacchetti provenienti dall'esterno non presentino indirizzi sorgente appartenenti a blocchi riservati o alla rete interna stessa. Blocchi da filtrare sistematicamente in ingresso dall'esterno:

- Indirizzi privati RFC 1918: `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`
- Loopback: `127.0.0.0/8`
- Multicast: `224.0.0.0/4` (se non utilizzato)
- Broadcast: `255.255.255.255/32`
- Indirizzi illegali: `0.0.0.0/8`

**Protocolli con negoziazione dinamica delle porte (es. FTP in modalità attiva).** Il protocollo FTP apre un *control channel* (TCP dalla porta client >1023 verso la porta 21 del server) su cui negozia una porta alta per il *data channel*. Il server avvia poi il data channel (dalla porta 20 verso la porta alta comunicata dal client). La porta del data channel non è nota a priori e viene trasmessa nel *payload* del pacchetto di controllo — non nell'header — rendendola invisibile al PF. Senza supporto applicativo, non è possibile scrivere una regola che autorizzi dinamicamente questo secondo canale.

**Attacchi data-driven.** Il PF non ispeziona il payload: non può distinguere una richiesta HTTP benigna (`GET /`) da una richiesta che trasporta un payload di SQL Injection (`GET /?q=' UNION SELECT ...`).

#### PF Stateful

Il PF tradizionale è **stateless**: decide su ogni pacchetto indipendentemente, senza memoria del traffico precedente. L'evoluzione **stateful** mantiene una tabella delle connessioni attive (connection tracking) e può autorizzare automaticamente i pacchetti di risposta di una connessione già stabilita, senza dover scrivere regole simmetriche esplicite. Questo è particolarmente utile per i protocolli senza connessione come UDP.

L'ulteriore evoluzione è il **Multilayer Protocol Inspection firewall**, che traccia l'intera storia della connessione verificando la coerenza del protocollo applicativo.

### 4.2 Application-Level Gateway (ALG)

Anche noto come **proxy server**, l'ALG è un "man-in-the-middle benevolo": si interpone tra client e server, agendo da server nei confronti del client e da client nei confronti del server reale. Comprende il protocollo applicativo e può quindi eseguire filtraggi di granularità molto più fine rispetto al PF.

**Vantaggi:** permette o nega specifici comandi applicativi; esamina la correttezza degli scambi protocollari; attiva dinamicamente regole sulla base della negoziazione client-server; si integra con processi esterni (antispam, antivirus, antimalware per web e posta); produce log dettagliatissimi delle sessioni.

**Svantaggi:** computazionalmente molto più pesante di un PF; specifico di un singolo protocollo applicativo; può richiedere configurazione esplicita del client (non sempre trasparente).

### 4.3 Circuit-Level Gateway (CLG)

Il CLG spezza la connessione a livello di trasporto: diventa endpoint del traffico, non un intermediario. Inoltrano il payload senza esaminarlo. Il suo utilizzo tipico è determinare quali connessioni originate dall'interno verso l'esterno sono ammissibili, differenziando le politiche sulla base degli utenti (quando integrato con un meccanismo di autenticazione).

**Vantaggi:** può essere trasparente per gli host fidati; agisce da intermediario generico senza conoscere i protocolli applicativi; si presta alla differenziazione per utente.

**Svantaggi:** filtraggio limitato a indirizzi, porte e utenti; richiede la modifica dello stack client o la configurazione consapevole delle applicazioni. Si combina efficacemente con un PF (per i dettagli di basso livello) e con un ALG (per i dettagli applicativi).

---

## 5. Collocazioni dei Firewall

**Bastion Host (BH).** Un sistema dedicato esclusivamente all'esecuzione di software firewall (tipicamente un ALG o un CLG). Il principio di robustezza impone che questo sistema offra la minima superficie di attacco possibile: nessun servizio superfluo, configurazione minimale, sistema operativo indurito.

**Personal Firewall.** Installato direttamente sulle singole macchine da proteggere, costituisce un'eccezione al principio della centralizzazione. Il vantaggio è la capacità di correlare il pacchetto con l'applicazione che lo ha generato o ricevuto, consentendo una granularità di controllo irraggiungibile a livello di rete. Lo svantaggio è la perdita della centralizzazione della configurazione e la tendenza degli utenti finali a configurarli in modalità "learning by doing" — ogni alert viene accettato acriticamente fino a rendere il firewall sostanzialmente inerte.

---

## 6. Topologie di Filtraggio

### 6.1 Topologia semplice

La configurazione più elementare è:

```
(rete esterna) ── [firewall] ── (rete interna)
```

Questa topologia non è adatta a reti che ospitano contemporaneamente **client** (che generano traffico uscente e devono essere completamente schermati dagli attacchi esterni) e **server** (che devono ricevere selettivamente traffico dall'esterno, ma non devono poter essere usati per attaccare i client interni se compromessi). La soluzione è la segmentazione in zone di rete con diversi livelli di fiducia.

### 6.2 Screened Single-Homed Bastion Host

Un packet filter router garantisce che solo il Bastion Host possa comunicare con l'esterno. Il BH implementa un ALG (eventualmente con autenticazione). Il filtraggio è doppio: a livello header (PF) e a livello applicativo (BH). Per ottenere il controllo completo della rete interna un attaccante deve compromettere entrambi i sistemi. Tuttavia, compromettere il solo PF può essere sufficiente per un accesso significativo — sebbene il PF sia tipicamente un sistema embedded con superficie di attacco ridottissima.

### 6.3 Screened Dual-Homed Bastion Host e DMZ

Il BH separa fisicamente due segmenti di rete distinti. Tra il PF esterno e la rete interna si crea una zona intermedia detta **DMZ** (*DeMilitarized Zone*): i server pubblici sono collocati in DMZ, separati sia dall'esterno (protetti dal PF) sia dalla rete interna (separati dal BH). La compromissione del solo PF esterno non dà accesso alla rete interna. Il limite è che tutto il traffico dei client interni deve attraversare il BH, anche quello del tutto innocuo.

### 6.4 Screened Subnet

La topologia più robusta utilizza **due PF router** che delimitano la DMZ su entrambi i lati:

```
(Internet) ── [PF esterno] ── (DMZ con server) ── [PF interno] ── (rete privata)
```

Questa configurazione offre i seguenti vantaggi aggiuntivi: rafforza la separazione tra esterno e interno; nasconde completamente all'esterno l'esistenza della subnet privata, ostacolando l'enumerazione da parte degli attaccanti; il PF interno può permettere al traffico "banale" di non passare dal BH.

### 6.5 Variazioni

Se si dispone di un PF molto affidabile o di budget limitato, è possibile unificare le funzioni dei due PF router in un singolo dispositivo con più di tre interfacce, creando anche più DMZ parallele (es. una per i server web, una per i server di posta). Al contrario, reti ad alta sicurezza possono concatenare in serie più DMZ con PF progressivamente più restrittivi.

---

## 7. Netfilter e nftables in Linux

### 7.1 Il framework Netfilter

**Netfilter** è il framework integrato nel kernel Linux che implementa il packet filtering. Il suo meccanismo fondamentale consiste nella definizione di **hook** — punti di intercettazione — in posizioni strategiche dello stack di rete del kernel. Ogni pacchetto che attraversa lo stack attiva gli hook corrispondenti al suo percorso. È possibile registrare programmi agli hook per eseguire controlli e manipolazioni sui pacchetti.

**nftables** è l'attuale strumento userspace per configurare il packet filter di Netfilter (il predecessore è **iptables**, ancora ampiamente diffuso). I concetti fondamentali sono gli stessi tra le due interfacce.

### 7.2 I Cinque Hook di Netfilter

Netfilter definisce cinque hook che coprono l'intero ciclo di vita di un pacchetto nel sistema:

| Hook | Momento di attivazione |
|---|---|
| `NF_IP_PRE_ROUTING` | Appena il pacchetto entra nello stack di rete, prima di qualsiasi decisione di instradamento |
| `NF_IP_LOCAL_IN` | Dopo l'instradamento, se il pacchetto è destinato a un processo locale |
| `NF_IP_FORWARD` | Dopo l'instradamento, se il pacchetto deve essere inoltrato a un altro host |
| `NF_IP_LOCAL_OUT` | Qualsiasi pacchetto generato localmente, appena entra nello stack di rete |
| `NF_IP_POST_ROUTING` | Qualsiasi pacchetto in uscita (locale o inoltrato), dopo l'instradamento e prima di lasciare il sistema |

Più programmi possono registrarsi allo stesso hook dichiarando un ordine di priorità: vengono invocati nell'ordine stabilito, e ognuno restituisce una decisione sul destino del pacchetto.

### 7.3 Il Percorso dei Pacchetti

Il percorso di un pacchetto attraverso gli hook dipende dalla sua origine e destinazione:

```
Pacchetto esterno in arrivo:
  PREROUTING → (routing decision) → LOCAL_IN   [destinato a processo locale]
                                 → FORWARD     [da inoltrate a host remoto]

Pacchetto generato localmente:
  LOCAL_OUT → POSTROUTING → [uscita verso la rete]

Tutti i pacchetti in uscita (locali o inoltrati):
  → POSTROUTING → [uscita verso la rete]
```

---

## 8. Tabelle, Catene e Regole

### 8.1 Tabelle

Le **tabelle** organizzano i controlli a seconda del tipo di decisione da prendere sul pacchetto. In iptables esistono tabelle predefinite; nftables non ha tabelle predefinite (vanno create al bisogno), ma il modello concettuale di iptables è utile come riferimento:

| Tabella | Funzione |
|---|---|
| `filter` | Tabella principale: decide se lasciare che un pacchetto prosegua o bloccarlo. È qui che si collocano le regole di firewall vere e proprie. |
| `nat` | Traduzione degli indirizzi di rete (NAT): modifica indirizzi sorgente o destinazione del pacchetto. |
| `mangle` | Modifica dell'header IP (es. TTL, TOS) e marcatura dei pacchetti per renderli riconoscibili da altre tabelle o strumenti. |
| `raw` | Meccanismo per contrassegnare pacchetti al fine di disabilitare il connection tracking (conntrack) per specifici flussi. |
| `conntrack` | Implementa automaticamente il riconoscimento delle connessioni e l'attribuzione dei pacchetti alle stesse (logica non configurabile direttamente dall'utente). |
| `security` | Imposta contrassegni di contesto di sicurezza SELinux sui pacchetti. |

> La tabella `filter` è quella su cui si lavora nella grande maggioranza dei casi. Le tabelle `nat` e `mangle` vengono usate per funzionalità più avanzate. È buona pratica usare DROP e ACCEPT **esclusivamente** nelle catene della tabella `filter`.

### 8.2 Catene

Le **catene** organizzano le regole in base all'hook a cui sono agganciate, determinando il *momento* del ciclo di vita del pacchetto in cui vengono valutate. In iptables le catene predefinite corrispondono direttamente agli hook di Netfilter:

| Catena iptables | Hook Netfilter | Uso tipico |
|---|---|---|
| `PREROUTING` | `NF_IP_PRE_ROUTING` | Regole da applicare appena il pacchetto entra nello stack; usata per DNAT |
| `INPUT` | `NF_IP_LOCAL_IN` | Regole sui pacchetti destinati a un processo locale del sistema |
| `FORWARD` | `NF_IP_FORWARD` | Regole sui pacchetti in transito (il sistema agisce da router) |
| `OUTPUT` | `NF_IP_LOCAL_OUT` | Regole sui pacchetti generati da processi locali |
| `POSTROUTING` | `NF_IP_POST_ROUTING` | Regole da applicare prima dell'uscita dal sistema; usata per SNAT/MASQUERADE |

Non tutte le tabelle registrano tutte le catene possibili. In nftables le catene non sono predefinite: vanno create manualmente specificando il tipo, l'hook a cui agganciarle e la priorità.

### 8.3 Regole

Le **regole** sono gli elementi costitutivi di una catena. Ogni regola esprime una condizione (o un insieme di condizioni) e un'azione da eseguire se tutte le condizioni sono soddisfatte. L'ordine delle regole in una catena è **fondamentale**: la scansione procede dall'alto verso il basso e la prima regola soddisfatta può terminare l'analisi.

Una catena si comporta come una sequenza di condizionali:

```
if   (pacchetto =~ condizione1) → azione1
elif (pacchetto =~ condizione2) → azione2
elif (pacchetto =~ condizione3) → azione3
...
else                            → default policy
```

Se un pacchetto non soddisfa le condizioni di alcuna regola (o soddisfa solo regole con azioni non definitive), la catena deve comunque restituire a Netfilter un risultato: questo è determinato dalla **default policy** della catena, che in un contesto di sicurezza dovrebbe essere `DROP`.

### 8.4 Azioni (Target/Verdict)

Le azioni si distinguono in due categorie:

**Terminating target / absolute verdict** — terminano la scansione della catena corrente e restituiscono il controllo a Netfilter:

| Target iptables | Statement nftables | Effetto |
|---|---|---|
| `ACCEPT` | `accept` | Netfilter prosegue l'analisi con le catene successive nel percorso del pacchetto |
| `DROP` | `drop` | Netfilter scarta il pacchetto silenziosamente (nessuna notifica al mittente) |
| `RETURN` | `return` | Torna alla catena chiamante, o applica la default policy se si è nella catena di base |

**Non-terminating target / statement** — eseguono un'azione sul pacchetto senza interrompere la scansione della catena:

| Target iptables | Statement nftables | Effetto |
|---|---|---|
| *(nessun target)* | `counter` | Incrementa contatori di pacchetti e byte ogni volta che un pacchetto soddisfa la regola; utile per il monitoraggio del traffico senza interferire con il transito |
| `LOG` | `log` | Il kernel registra i dettagli del pacchetto nel syslog; il pacchetto continua la scansione delle regole successive |

> La separazione tra `LOG` e `DROP` è intenzionale e potente: si può scrivere una coppia di regole consecutive — prima un `LOG` (non-terminating), poi un `DROP` (terminating) — per registrare e scartare contemporaneamente una categoria di pacchetti, senza duplicare le condizioni di match.

---

## 9. Configurazione con nftables: Struttura di Base

nftables introduce una sintassi unificata e più coerente rispetto a iptables. I concetti di tabella, catena e regola rimangono, ma nessuno è predefinito: tutto va creato esplicitamente.

```bash
# Visualizzare la configurazione attuale
nft list ruleset

# Struttura di base di un ruleset nftables (sintassi dichiarativa)
table inet filter {
    chain input {
        type filter hook input priority 0; policy drop;
        
        # Permette traffico loopback
        iifname "lo" accept
        
        # Permette connessioni già stabilite e correlate (stateful)
        ct state established,related accept
        
        # Permette ICMP (ping)
        ip protocol icmp accept
        
        # Permette SSH in ingresso
        tcp dport 22 accept
        
        # Tutto il resto viene scartato (default policy: drop)
    }
    
    chain forward {
        type filter hook forward priority 0; policy drop;
    }
    
    chain output {
        type filter hook output priority 0; policy accept;
    }
}
```

Principi guida nella scrittura del ruleset:

- Definire sempre la **default policy** per ciascuna catena. Per `INPUT` e `FORWARD` la policy deve essere `drop` (default deny).
- Autorizzare il traffico di **loopback** (`lo`) prima di tutto il resto.
- Autorizzare esplicitamente le connessioni `established` e `related` per permettere le risposte al traffico già autorizzato in uscita (conntrack stateful).
- Applicare le regole più specifiche e frequenti nelle prime posizioni della catena, per ottimizzare le prestazioni.
- Prefissare sempre un `log` alle regole `drop` critiche per mantenere visibilità sugli eventi di sicurezza.

---

## Riepilogo dei Concetti Chiave

| Concetto | Definizione Sintetica |
|----------|-----------------------|
| **Firewall** | Architettura di difesa perimetrale che impone un punto di passaggio obbligato al traffico tra zone di rete |
| **Default deny** | Politica che scarta tutto il traffico non esplicitamente autorizzato |
| **Ingress/Egress filtering** | Filtraggio del traffico entrante / uscente dalla rete protetta |
| **Packet Filter (PF)** | Firewall che esamina solo gli header dei pacchetti e applica regole condizionali |
| **PF Stateful** | PF con connection tracking: autorizza automaticamente i pacchetti di risposta di connessioni stabilite |
| **Application-Level Gateway (ALG)** | Proxy che comprende il protocollo applicativo e opera da man-in-the-middle benevolo |
| **Circuit-Level Gateway (CLG)** | Firewall che spezza la connessione a livello di trasporto, senza esaminare il payload |
| **Bastion Host** | Sistema dedicato all'esecuzione di software firewall, con superficie di attacco minimale |
| **DMZ** | Zona di rete intermedia tra esterno e rete privata, dove si collocano i server pubblici |
| **Screened Subnet** | Topologia con due PF che delimitano la DMZ, massimizzando la separazione tra esterno e interno |
| **Netfilter** | Framework del kernel Linux che definisce hook nello stack di rete per intercettare i pacchetti |
| **Hook** | Punto di intercettazione nello stack di rete (PREROUTING, INPUT, FORWARD, OUTPUT, POSTROUTING) |
| **nftables** | Strumento userspace per configurare il packet filter di Netfilter (successore di iptables) |
| **Tabella** | Organizzazione delle regole per tipo di funzione (filter, nat, mangle, raw, …) |
| **Catena** | Sequenza ordinata di regole agganciata a uno specifico hook di Netfilter |
| **Regola** | Elemento costitutivo di una catena: condizione → azione |
| **Default policy** | Azione applicata a un pacchetto che non ha soddisfatto nessuna regola della catena |
| **ACCEPT / DROP** | Azioni terminanti: prosegue il percorso del pacchetto / scarta il pacchetto silenziosamente |
| **LOG** | Azione non terminante: registra i dettagli del pacchetto nel syslog senza interrompere la scansione |
| **conntrack** | Meccanismo di connection tracking del kernel che permette il PF stateful |
