# Laboratorio di Sicurezza Informatica
## Rischi, Attacchi e Difese
**Prof. Marco Prandini** — Dipartimento di Informatica – Scienza e Ingegneria  
Alma Mater Studiorum – Università di Bologna

---

## 1. Il Rischio Cyber

La gestione della sicurezza informatica è sostanzialmente un **esercizio di gestione del rischio**, definito come:

> *"il potenziale danno immateriale, perdita economica, o distruzione di risorse che risulterebbe da un evento (malevolo)"*

### Formula del Rischio

$$\text{RISCHIO} = \text{PROBABILITÀ} \times \text{IMPATTO}$$

**Esempio:** probabilità del 4% di subire un danno di 15.000 € in un anno → rischio = **600 €/anno**

### Implicazioni operative

Per **gestire** il rischio occorre:
- Valutare la probabilità di ogni evento potenzialmente dannoso
- Quantificare l'impatto di ogni possibile azione malevola

Per **mitigare** il rischio si progettano e implementano **contromisure** (che devono essere convenienti), di cui è necessario valutare l'efficacia in termini di riduzione della probabilità degli eventi dannosi e/o del loro impatto.

> *"Progress just means bad things happen faster."* — Terry Pratchett  
> *"If you think technology can solve your security problems, then you don't understand the problems and you don't understand the technology."* — Bruce Schneier

---

## 2. Il Panorama delle Minacce (ENISA Threat Landscape 2023)

Fonte: [https://www.enisa.europa.eu/publications/enisa-threat-landscape-2023](https://www.enisa.europa.eu/publications/enisa-threat-landscape-2023)

### Distribuzione per tipo di minaccia

| Minaccia | Incidenti | % |
|---|---|---|
| Ransomware | 1.48K | 31.32% |
| DDoS | 1.01K | 21.40% |
| Data (violazioni dati) | 0.95K | 20.09% |
| Malware | 0.39K | 8.24% |
| Social Engineering | 0.37K | 7.88% |
| Information Manipulation | 0.23K | 4.81% |
| Web Threats | 0.14K | 3.03% |
| Supply Chain Attack | 0.10K | 2.10% |
| Zero Day | (residuo) | — |

### Motivazioni degli attaccanti

| Motivazione | Prevalenza |
|---|---|
| Financial Gain | ~29% (prima causa per ransomware) |
| Disruption | ~18% (prima causa per DDoS) |
| Unknown | ~4% |
| Espionage | ~2% |
| Ideology | ~2% |
| Destruction | (residuale) |

### Bersagli principali

| Settore | Incidenti | % |
|---|---|---|
| Public Administration | 991 | 19% |
| Targeted Individuals | 568 | 11% |
| Health | 406 | 8% |
| Digital Infrastructure | 348 | 7% |
| Manufacturing | 339 | 7% |
| Transport | 319 | 6% |
| Digital Service Provider | 304 | 6% |
| Banking/Finance | 299 | 6% |

---

## 3. Esempi di Attacchi

### 3.1 Interruzione del Servizio (DoS / DDoS)

**Obiettivo:** violare la **"A"** della triade CIA (Availability).

- **Denial of Service (DoS):** traffico o carico di calcolo insostenibile generato verso la vittima, spesso senza strumenti sofisticati (forza bruta)
- **Distributed DoS (DDoS):** l'attaccante si avvale di un'armata di computer compromessi (**botnet**), di per sé di poco valore ma numericamente sufficienti a creare grande forza d'urto

**Architettura tipica:** Attacker → Bot coordinator → Botnet → Victim

### 3.2 Furti di Identità

Sottrazione di informazioni personali (es. codice fiscale) per creare nuovi account, effettuare acquisti o commettere frodi.

**Casi notevoli (per numero di record esposti):**
- Yahoo: 1 miliardo (2016) + 500 milioni (2016)
- Equifax: 143 milioni (2017)
- Heartland Payment Sys.: 130 milioni (2009)
- LinkedIn: 117 milioni (2016)

### 3.3 Phishing

E-mail, messaggi di testo, telefonate o siti web fraudolenti progettati per indurre gli utenti a:
- Scaricare malware
- Condividere informazioni sensibili o dati personali
- Intraprendere azioni che espongono sé stessi o le proprie organizzazioni alla criminalità informatica

**Dati di rilievo:**
- Seconda causa più comune di violazione dei dati
- Costo medio per le vittime: **4,91 milioni di dollari**
- Google individua **300.000 nuovi siti di phishing al mese**, molti online per meno di un'ora

**Ciclo dell'attacco:**
1. L'attaccante invia la mail di phishing al target
2. La vittima clicca sul link e visita il sito falso
3. L'attaccante raccoglie le credenziali inserite
4. L'attaccante le utilizza per accedere alle informazioni private

**Indicatori di phishing da riconoscere:**
- Errori ortografici, di punteggiatura e grammaticali
- Link che puntano a siti sospetti (URL non corrispondente al mittente)
- Richieste di cliccare link per verificare l'account (le organizzazioni legittime non lo fanno)
- Mittente con dominio apparentemente attendibile ma non verificato

### 3.4 Ransomware

**Meccanismo:** cifratura dei dati critici (file, database, applicazioni) della vittima; il ripristino è subordinato al pagamento di un riscatto.

**Caso emblematico:** WannaCry — attacco globale che ha colpito l'NHS britannico e aziende in Spagna, Russia, Ucraina e Taiwan.

### 3.5 Ransomware as a Service (RaaS)

Modello di business in cui lo **sviluppatore del malware** (es. DarkSide) eroga il servizio ad **affiliati** (cracker) che non necessariamente possiedono le competenze tecniche per creare il ransomware, ma sono in grado di penetrare nel sistema della vittima.

**Il servizio include:**
- Supporto tecnico per i cracker
- Negoziazione con le vittime
- Campagne di pressione su misura

**Struttura tariffaria di DarkSide:**
- 25% per riscatti < 500.000 $
- Sconti progressivi fino al 10% per riscatti > 5 milioni $

**Flusso dell'attacco RaaS:**
1. Developer fornisce exploit code all'Affiliate
2. Affiliate aggiorna il codice su Hosting/Exploit
3. La vittima riceve il link via Email/Web
4. La vittima scarica e installa il malware
5. I dati vengono esfiltrati verso l'Affiliate
6. La vittima paga il riscatto
7. Il riscatto transita attraverso un Money Launderer
8. Developer e Launderer ricevono le rispettive quote

### 3.6 Crypto-jacking

**Meccanismo:** uso non autorizzato dei dispositivi delle vittime per estrarre criptovaluta. Funziona incorporando uno script JavaScript in un sito web compromesso: ogni visitatore esegue solo una piccola quota di mining, ma il totale del tempo di calcolo aggregato genera valore reale per l'attaccante.

### 3.7 Supply Chain Attack

**Meccanismo:** sfrutta l'interdipendenza del mondo moderno colpendo un bersaglio intermedio (fornitore di componenti hardware o software) per raggiungere le vere vittime.

**Esempi:**
- **Target** (2014): violazione tramite una società HVAC fornitrice
- **SolarWinds** (2020): backdoor nella piattaforma Orion

### 3.8 Attacchi "alla vecchia maniera"

Il **furto fisico** di un dispositivo resta uno strumento valido:
- Salta le difese predisposte a controllare il traffico di rete
- Salta le protezioni offerte dal sistema operativo
- Causa immediata perdita di Confidenzialità e Disponibilità

Anche la **distruzione fisica** minaccia la Disponibilità.

**Elemento particolarmente a rischio: lo smartphone**
- Contiene dati di immediato valore
- Rappresenta il punto privilegiato per regolare l'accesso a tutti i dati e servizi esterni (app di autenticazione, SMS di conferma, ecc.)

---

## 4. Incidenti Notevoli

### 4.1 Cronologia 2020 (selezione)

| Categoria | Esempi notevoli |
|---|---|
| Data Breach | EasyJet (9M record), CAM4 (10M+ utenti), Marriott (multa 18M£ per leak 339M clienti 2014-2018) |
| Ransomware | Travelex (6M$), Garmin (3 gg down), CWT Travel (4.5M$), ERT, Software AG (20M$) |
| Cyberwar | Stuxnet, backdoor in software fiscale per aziende straniere in Cina, attacchi infrastrutture idriche IL |

### 4.2 L'Effetto COVID

Il lockdown ha determinato:
- **Telelavoro** → maggiore utilizzo di dispositivi e reti non gestiti
- **Incremento dei servizi online** → e-commerce, e-banking, social network, con nuovi utenti non consapevoli dei rischi

**Impatto statistico (Fonte: Fintech News):**
- +238% attacchi cyber alle banche
- +600% attacchi phishing (da fine febbraio)
- +148% attacchi ransomware (marzo); pagamento medio +33% → **111.605 $** rispetto al Q4 2019

### 4.3 Cronologia 2022 (selezione)

- **Gennaio:** attacco cyber su larga scala contro siti governativi ucraini; hackeraggio di TV e radio iraniane
- **Febbraio:** ransomware su terminal portuali (Belgio, Germania, Paesi Bassi); DDoS sulle forze armate ucraine
- **Marzo:** furto di 2 certificati di code signing NVIDIA; violazione Kremlin.ru
- **Maggio:** Costa Rica perde 200M$ da ransomware; chiusura del Lincoln College (157 anni di storia) in Illinois; record DDoS su banca russa (450 GB/sec)
- **Giugno:** record HTTPS DDoS su infrastruttura Cloudflare
- **Luglio:** data breach Twitter (5.4M account)
- **Agosto:** Cisco violata da gruppo ransomware; breach maggiore acquedotto UK (5TB)
- **Settembre:** data breach Uber
- **Ottobre:** dati personali 10M australiani rubati; 10M cittadini australiani esposti
- **Dicembre:** campagna Android per furto credenziali Facebook (300K utenti, 71 paesi)

### 4.4 Fast Forward: Attacchi Notevoli 2023–2025

| Data | Target | Descrizione | Impatto |
|---|---|---|---|
| Luglio 2025 – Detroit, U.S. | Great Lakes Water Authority | Violazione sistema di monitoraggio | Rischio operativo, indagini in corso |
| Aprile 2025 – Bremanger, Norvegia | Diga | Controllo delle paratoie, ~500 l/s per ~4 ore | Cyber-sabotaggio infrastrutture idriche |
| Dicembre 2024 – Danimarca | Water utility plant | Attacco distruttivo | Danni limitati |
| Agosto 2024 – Halliburton | Settore energia | Accesso non autorizzato ai sistemi | Potenziale impatto su sistemi industriali |
| Gennaio 2024 – Lviv, Ucraina | Heating utility | Malware FrostyGoop: riscaldamento disabilitato ~600 edifici per ~48h | Rischio per la popolazione |
| 2023–2024 – U.S. | Critical infrastructure | Accesso a ICS in settori idrici e alimentari | Potenziale danno fisico |

### 4.5 Guerra Cyber

Nel 2009 il Segretario alla Difesa USA Robert Gates ha dichiarato che il **cyberspazio è il "quinto dominio"** delle operazioni militari, accanto a terra, mare, aria e spazio. Gli USA schierano attualmente **6.200 cyber-soldati**.

**Principali attori statali:** North Corea, Russia, Cina, Iran.

**Caso Stuxnet** — worm a scopo militare contro il programma nucleare iraniano:
1. **Infection:** entra tramite USB, infetta macchine Windows con certificato digitale falsificato
2. **Search:** verifica se il sistema appartiene all'ICS Siemens bersaglio
3. **Update:** se è il target, si aggiorna via Internet
4. **Compromise:** compromette i controllori logici sfruttando zero-day
5. **Control:** spia le operazioni, poi prende il controllo delle centrifughe
6. **Deceive and Destroy:** invia feedback falsi ai controllori esterni mentre porta le centrifughe al fallimento

### 4.6 SolarWinds (Supply Chain)

SolarWinds fornisce servizi a: esercito USA, Pentagono, Dipartimento di Stato, NASA, NSA, USPS, Ministero di Giustizia, Ufficio del Presidente USA, le 5 principali aziende di accounting — e Telecom Italia.

**Falla nella piattaforma Orion:**
- Accesso senza autenticazione
- Visibilità di tutti i dati dei clienti

**Elemento chiave: persistenza attraverso la supply chain**  
Anziché cercare di persistere in ciascun sistema vittima, gli attaccanti hanno violato il **processo di aggiornamento delle librerie** usate da SolarWinds:
1. Codice malevolo inserito silenziosamente nella DLL di SolarWinds
2. L'utente installa l'app SolarWinds compromessa tramite aggiornamento software
3. La DLL malevola contatta l'infrastruttura C2 per ricevere comandi e payload aggiuntivi

### 4.7 Dependency Confusion (Supply Chain su Repository)

*(Ars Technica, 16 febbraio 2021)*

Un ricercatore ha inserito codice malevolo su repository pubblici (NPM per JavaScript, PyPI per Python) **con lo stesso nome dei pacchetti privati** usati da Apple, Microsoft, Tesla e altre 33 aziende, inducendole a scaricare e installare automaticamente il codice contraffatto.

Il fenomeno si ripete sugli **app store**, colpendo normali utenti di smartphone.

### 4.8 Backdoor xz/liblzma (Open Source Social Engineering)

**29 marzo 2024:** scoperta backdoor nella libreria `xz/liblzma` (algoritmi di compressione usati da comandi e servizi Linux) — potenziale **Remote Code Execution (RCE)**.

- CVE: [CVE-2024-3094](https://www.cve.org/CVERecord?id=CVE-2024-3094)

**Due aspetti notevoli:**

**1. Sofisticazione tecnica** elevata (analisi: https://x.com/fr0gger_/status/1774342248437813525)

**2. Attacco di social engineering nel processo di sviluppo OSS** — timeline:

| Anno | Evento |
|---|---|
| 2021 | *Jia Tan* crea il proprio account GitHub |
| 2022 | *Jia Tan* propone una patch; compare *Jigar Kumar* che insiste per l'accettazione |
| 2022 | *Jigar Kumar* fa pressioni perché il maintainer *Lasse Collin* si faccia affiancare, sfruttandone i problemi mentali |
| 2022+ | *Jia Tan* diventa "affidabile" e accumula numerosi contributi al progetto |
| — | *Jigar Kumar* scompare; compare *Dennis Ens* a fare pressioni per la sostituzione di Collin |
| 2023 | *Jia Tan* sostituisce *Lasse Collin*; approva immediatamente un contributo di *Hans Jansen* (fino a quel momento quasi inattivo) |
| 2023 | *Tan* e *Jansen* fanno pressioni per l'inclusione della loro versione in Debian |
| 2024 | *Tan* sposta il progetto su pagine sotto il suo controllo; carica il componente finale per attivare la backdoor |

---

## 5. Difesa

### 5.1 La Difesa come Processo Continuo

> *"Sicurezza non è valutare la situazione presente e comprare un **prodotto**, bensì definire un **processo** per tenere traccia delle continue evoluzioni dei rischi e dell'efficacia delle contromisure."*

Il ciclo di vita della sicurezza comprende le fasi: **Understand → Assess → Design → Accept → Implement → Certify → Validate → Accredit → Go Live → Operate & Maintain → Respond → Review → Retire** (e riparte da Understand).

### 5.2 Il Metodo: Framework e Certificazioni

La messa in sicurezza deve essere un **processo metodico**. I framework aiutano nella sistematizzazione; le certificazioni forniscono evidenza di terza parte dell'adozione di misure efficaci (la **sicurezza della supply chain** è diventata elemento cruciale).

**NIST Cybersecurity Framework** ([https://www.nist.gov/cyberframework](https://www.nist.gov/cyberframework)) — 5 funzioni:

| Funzione | Sottocategorie principali |
|---|---|
| **1. Identify** | Asset Management (AM), Business Environment (BE), Governance (GV), Risk Assessment (RA), Risk Management Strategy (RM) |
| **2. Protect** | Access Control (AC), Awareness Training (AT), Data Security (DS), Information Protection (IP), Processes and Procedures (PP), Protective Technology (PT) |
| **3. Detect** | Anomalies and Events (AE), Security Continuous Monitoring (CM), Detection Processes (DP) |
| **4. Respond** | Response Planning (RP), Communications (CO), Analysis (AN), Mitigation (MI), Improvements (IM) |
| **5. Recover** | Recovery Planning (RP), Improvements (IM), Communications (CO) |

### 5.3 Politiche e Meccanismi

- **Security Policy:** dichiarazione di ciò che è consentito o proibito
- **Security Mechanism:** metodo, strumento o procedura per far rispettare una politica

> Nota: i meccanismi non sono necessariamente tecnici; tra i più importanti figurano **comportamenti e regole di interazione tra persone**.

### 5.4 Obiettivi dei Meccanismi

I meccanismi combinano diverse strategie:

| Strategia | Descrizione |
|---|---|
| **Prevenzione** | L'attacco deve fallire; meccanismi invasivi e non aggirabili |
| **Rilevazione** | L'attacco può avere successo ma deve essere notato e riportato (inefficace per alcune minacce, es. disclosure) |
| **Reazione** | L'attacco rilevato viene mitigato per ridurne gravità o estensione |
| **Ripristino** | Le conseguenze vengono ridotte o azzerate, ripristinando le proprietà di sicurezza violate |

### 5.5 Superficie di Attacco

Politiche e meccanismi si applicano a ogni interazione del sistema col mondo esterno (o tra sottosistemi).

- **Vettore di attacco:** ogni modo reso accessibile a un attaccante per stimolare un'interazione
- **Canali di accesso:** Fisico, Cyber (remoto via cavo o wireless), Umano
- **Superficie di attacco:** l'insieme di tutti i vettori

### 5.6 Prevenzione — Security Engineering

**Prima sfida:** non trascurare nessun dettaglio → inventario di tutti i componenti fisici e catalogo di tutti i servizi.

**Raccolta dei requisiti** — molto diversa da quella tradizionale: focalizzata sul **"non deve accadere"** invece che sul "deve funzionare".

Strumenti metodologici:

| Strumento | Scopo |
|---|---|
| **Misuse Case** | Verificare se una minaccia si può concretizzare in un attacco |
| **Security Use Case** | Distillare i requisiti dei meccanismi da applicare |
| **Attack Tree** | Modellare come si combinano diversi possibili eventi e condizioni al contorno |

**Relazioni tra concetti:** Assets & Services → (vulnerabili a) Security Threats → (necessitano) Security Requirements → (richiedono) Security Mechanisms. I Misuse Cases alimentano i Security Use Cases, che guidano i Security Requirements; entrambi contrastano le minacce.

### 5.7 Security Engineering — Migliori Pratiche

1. Basare le decisioni su una **politica esplicita** (identity, access control, content-specific, network/infrastructure, regulatory, advisory)
2. **Defense in depth** — evitare singoli punti di fallimento
3. **Fail secure** — fallire in modo certo
4. Bilanciare sicurezza e usabilità
5. Essere consapevoli dell'**ingegneria sociale**
6. Usare **ridondanza e diversità** per ridurre i rischi
7. **Validare tutti gli input**
8. **Compartimentare** i beni
9. Progettare per il **deployment**
10. Progettare per il **ripristino**

### 5.8 Prevenzione — Testing

**Obiettivo:** verificare la presenza di vulnerabilità e l'esposizione a rischi nuovi rispetto al momento della progettazione.

**Problema concettuale:** non si può dimostrare l'assenza di problemi; è possibile solo tentare di sollecitare il sistema nel modo più completo possibile.

**Tre livelli di approfondimento:**

| Livello | Descrizione |
|---|---|
| **Vulnerability Assessment** | Identificazione sistematica delle vulnerabilità note |
| **Penetration Testing** | Tentativo controllato di sfruttare le vulnerabilità identificate |
| **Red Team Operations** | Simulazione completa di un attaccante reale, con tecniche avanzate e obiettivi definiti |

**Quando testare?** — **Prima è meglio** (*Shift Left Testing*)

Il principio del Shift Left raccomanda di anticipare il testing il più possibile nel ciclo di sviluppo (SDLC):
- Testing del nuovo codice (fase di sviluppo)
- Testing ad ogni build
- Testing ad ogni deployment
- Testing in produzione

Anticipare il testing riduce drasticamente il costo di correzione delle vulnerabilità.

---

*Appunti elaborati a partire dalle slide del corso DM895 — A.A. 2024/2025*
