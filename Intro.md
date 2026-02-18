# Laboratorio di Sicurezza Informatica — Lezione Introduttiva

**Docente:** Prof. Marco Prandini
**Dipartimento:** Informatica – Scienza e Ingegneria, Alma Mater Studiorum – Università di Bologna

---

## 1. La Sicurezza Informatica ci Riguarda?

Sì, ben prima che come professionisti. Nelle nostre vite quotidiane, tre categorie di elementi sono ormai informatizzati, irrinunciabili e in molti casi, se danneggiati, **insostituibili** (in assoluto o in tempo utile per evitare conseguenze gravi):

- **Infrastrutture critiche** per la "civiltà"
- **Sistemi di comunicazione** ed elaborazione delle informazioni
- **Archivi di informazioni personali**

### Definizione operativa

**Sicurezza informatica** è tutto ciò che ha a che fare col contrasto di **azioni deliberate** che provochino danni. Termini diversi hanno sfumature specifiche, ma sono spesso usati "popolarmente" in modo intercambiabile: sicurezza dell'informazione, IT security, cybersecurity, ecc.

> **Distinzione chiave:** Si usa "sicurezza" nel senso del termine inglese ***security*** (contrasto di azioni deliberate), ricordando che in italiano il termine copre anche il contrasto di eventi accidentali (in inglese tradotto ***safety***).

---

## 2. Impatto Sociale della Cyber(in)security

La slide presenta una rassegna di incidenti reali che dimostrano la gravità delle minacce informatiche alle infrastrutture critiche:

- **2020** — Donna deceduta durante un attacco ransomware a un ospedale tedesco *(The Verge, Sep 17, 2020)*
- **2000** — Maroochy waste management
- **2008** — Refahiye pipeline
- **2018** — Saudi Chemical Company
- **2020** — Natanz "stuxnet 2"

Ulteriori casi citati nella mappa mondiale degli attacchi alle infrastrutture energetiche:

- **2013** — Attacco coordinato in Northern California ($15M di danni a linee e trasformatori di sottostazione)
- **2014** — Compromissione di utility USA via brute-force del portale internet
- **2014** — "RedHack" contro compagnia elettrica USA (multa $2.7M, $650K di bollette cancellate)
- **2014** — Operazione "Cleaver": penetrazione in 50+ aziende mondiali
- **2014** — Dragonfly/Energetic Bear: campagna di cyber-spionaggio contro operatori di rete e generazione elettrica (Medio Oriente)
- **2015** — 225.000 cittadini ucraini senza corrente per attacco a 3 compagnie energetiche
- **2016** — Secondo attacco alla rete ucraina (Crash Override malware, comunicazione diretta con ICS)
- **2017** — Infiltrazione di sistemi di sicurezza critici in impianti nucleari, petroliferi e del gas
- **2017** — Phishing contro Electricity Supply Board irlandese per infiltrare sistemi di controllo industriale
- **2017** — Virtual wire tap su operatore telecom UK (traffico non cifrato, Northern Ireland and Wales)
- **2018** — DHS: intrusione multi-stadio in reti del settore energia USA (spear phishing + lateral movement)
- **2010** — Stuxnet: centrifughe di arricchimento dell'uranio iraniano fuori controllo
- **2011** — Night Dragon: furto di informazioni proprietarie e riservate da compagnie oil & gas in Asia
- **2012** — Shamoon virus: 30.000 computer e hard drive distrutti in compagnia energetica del Medio Oriente (variante più distruttiva nel 2016-2017)

*(Fonte infografica: "Hackers are causing blackouts. It's time to boost our cyber resilience." — World Economic Forum, Mar 27, 2019)*

---

## 3. Impatto Economico della Cyber(in)security

> *"Se il cybercrime fosse una nazione, sarebbe nel G3, con un GDP >10T$ previsto per il 2025"*
> *(Nota del docente: titolo un po' sensazionalistico — è la stima peggiore dei danni causati, non dei guadagni, e circolano anche cifre minori. Comunque, con una crescita prevista annua del 10-15%, ci arriverà...)*

### Il cybercrime come business criminale in crescita

- Più **lucrativo** del mercato mondiale della droga
- Più **dannoso** di tutti i disastri naturali cumulati (per ora...)

### Un modello criminale attrattivo

- Utilizzabile in innumerevoli settori
- A basso rischio (**0,05%**) di individuazione e prosecuzione legale

### Ma anche un mercato del lavoro lecito

Alimentato da investimenti ingenti per la difesa:

- Cresciuto del **12-15%** all'anno negli ultimi anni
- **2018-2021:** totalizzati **1.000 miliardi $**
- **Solo nel 2025:** spesi **300 miliardi di US$**

---

## 4. Proprietà di Sicurezza — La Triade CIA

> **Annotazione manoscritta:** *Confidentiality, Integrity, Availability*

La ==sicurezza di un sistema== può essere ==scomposta in== tre proprietà chiave, riassunte dalla sigla **CIA**:

### Confidentiality (Riservatezza)

Mantenere inaccessibili ==dati==, o proprietà di un sistema, ==a chi non sia autorizzato== a conoscerli.

### Integrity (Integrità)

Poter garantire che ==il contenuto== e/o ==l'origine di un dato corrispondano== ==a quanto si ritiene corretto==.

### Availability (Disponibilità)

Poter garantire la ==possibilità effettiva== di accedere a ==dati e servizi quando necessario==.

---

## 5. Le Minacce e gli Attacchi

> **Annotazione manoscritta:** *THREAT*

### Definizioni fondamentali

- **Minaccia (threat):** una ==condizione che potenzialmente può compromettere una o più delle proprietà di sicurezza==. Esiste indipendentemente dal fatto che venga concretizzata.
- **Attacco (attack):** l'azione che porta al concretizzarsi di una minaccia.
- **Attaccante (attacker):** l'entità che sferra l'attacco.

### Tassonomia dei potenziali attaccanti

Le minacce sono indissolubilmente legate alle **intenzioni** dei potenziali attaccanti (in ordine crescente di sofisticazione e risorse):

1. Script kiddies
2. Criminali comuni
3. Insider disonesti e impiegati vendicativi
4. Reporter
5. Ricercatori
6. Attivisti
7. Criminali organizzati
8. Spie industriali
9. Governi ed eserciti

---

## 6. Tipologie di Attacchi e Minacce

| Categoria | Proprietà violata | Tecniche |
|---|---|---|
| **Disclosure** | Riservatezza | Snooping, Eavesdropping |
| **Deception** | Integrità (porta ad accettare dati falsi) | Modification, Alteration, Masquerading, Spoofing, Repudiation of origin, Denial of receipt |
| **Disruption** | Disponibilità | Delay, Destruction, Denial of service |
| **Usurpation** | Uso non autorizzato del sistema | *(combinazione di tecniche precedenti)* |

---

## 7. Vulnerabilità ed Exploit

> **Annotazione manoscritta:** *VULNERABILITY, EXPLOIT*
> **Nota sulle slide:** *"Termini che saranno definiti in seguito"* (riferito a "politiche" e "meccanismi")

### Politiche e meccanismi

Se le ==politiche== e i ==meccanismi== di protezione di un sistema fossero perfetti, le minacce non potrebbero concretizzarsi: neutralizzano i **vettori di attacco**.

### Quando gli attacchi hanno successo

Gli attacchi hanno successo se esistono **errori**:

- Nell'individuazione della **superficie di attacco** (==porosità== — un vettore esiste là dove non dovrebbe)
- Nella definizione di una politica o nell'implementazione di un meccanismo → **vulnerabilità (vulnerability)**:
  - Può essere ==strutturale nell'hardware o software==
  - Può ==dipendere dalla configurazione==
  - Può ==dipendere da un uso scorretto==

### Exploit

Uno ==strumento per trarre vantaggio da una vulnerabilità== concretizzando una minaccia:

- **Tecnico** (cracking)
- **Umano** (social engineering)

---

## 8. Come Avviene un Attacco

I ==modi di entrare in un sistema sono vari== e ==spesso usano== come anello debole della catena ==l'essere umano==.

==Spesso l'obiettivo è installare qualche tipo di **malware**== (software malevolo):

| Tipo di malware | Funzione |
|---|---|
| **RAT** (Remote Access Trojan) | ==Consente accesso== al sistema |
| **Cryptolocker / Ransomware** | ==Rende il sistema inutilizzabile== |
| **Spyware** | ==Esporta verso l'attaccante i dati== della vittima |
| **Botnet agent** | ==Utilizza la vittima per effettuare altri attacchi== |

> **Modello di riferimento:** *The Unified Kill Chain* (Pols, 2017 — Cyber Security Academy), che articola l'attacco in tre macro-fasi circolari: **Initial Foothold** (Compromised System) → **Network Propagation** (Internal Network) → **Action on Objectives** (Critical Asset Access), attraverso stadi quali Reconnaissance, Weaponization, Delivery, Social Engineering, Exploitation, Persistence, Defense Evasion, Command & Control, Pivoting, Discovery, Privilege Escalation, Execution, Credential Access, Lateral Movement, Target Manipulation, Collection, Exfiltration.

---

## 9. Obiettivi del Corso

**Ethical hacking** — il processo in cui si adotta lo stesso approccio che impiegherebbero attaccanti malevoli, ma in un contesto **controllato tecnicamente e legalmente**, per scoprire le vulnerabilità prima che lo facciano loro:

- Per **valutare il livello di esposizione e di rischio**
- Per **comprendere meglio come progettare le contromisure**

---

## 10. Cosa Faremo in Questo Corso

### Finalità

- Panoramica dei principali problemi pratici della sicurezza informatica
- Acquisizione delle basi delle **tecniche offensive** per la valutazione delle vulnerabilità e delle **tecniche difensive** per la loro mitigazione

### Struttura

**6 crediti / 60 ore** divise in circa **25 ore di teoria + 35 ore di pratica**.

### Percorso tematico

1. **Accesso illecito ai sistemi:** come si ottiene, si rileva, si blocca, e si aggirano i blocchi
2. **Dall'interno:** metodi di privilege escalation e strumenti per rilevare ed eliminare i percorsi possibili
3. **Attacchi al traffico in rete**
4. **Crittografia** e suo utilizzo per difendere i dati utente e le comunicazioni

---

## 11. Programma Dettagliato

### Modulo 1 — Accesso ai Sistemi

- Offensive security, Vulnerability assessment e penetration testing
- I primi passi della Kill Chain ed Enumerazione
- **Vettori di attacco:**
  - Web: SQL injection, path traversal e simili
  - Binari: buffer overflow e suo sfruttamento
- **Protezione degli host con dispositivi di rete:** configurazione di packet filtering firewall
- **Modelli di intrusion detection:**
  - Host-based IDS: event management
  - Analisi del traffico e network-based IDS
- Back to offensive: modi per scavalcare firewall e IDS
- Sicurezza fisica
- Supply chain / repo poisoning

### Modulo 2 — Dall'Interno

- Modelli di controllo dell'accesso e loro utilizzo per aumentare i privilegi
- Permessi Linux e NTFS
- Bit speciali, Capabilities, ACL
- Privilegi di amministrazione
- Gestione dei processi
- Strategie di occultamento e persistenza
- Rilevare le alterazioni al sistema
- Host-based IDS: integrity checking

### Modulo 3 — Attacchi al Traffico

- Sniffing
- Spoofing
- Hijacking
- DDoS (usare la rete per abbassare i sistemi)
- Contesti: LAN, WLAN, Internet

### Modulo 4 — Crittografia e Applicazioni

- Crittografia simmetrica
- Funzioni hash e robustezza delle password (dictionary attacks, brute forcing, linee guida per la scelta)
- Crittografia asimmetrica e protezione del traffico
- Autenticazione attiva
- Accesso sicuro ai sistemi (SSH)
- Public-key Infrastructure e TLS
- IPSEC e altre VPN, TOR

---

## 12. Informazioni Logistiche

### Orario

**15 settimane nette × 5 ore:**

- **3 ore** il mercoledì in **LAB4**
- **2 ore** il venerdì in **aula 0.5**

### Sospensioni previste

- **Mercoledì 4 marzo** — nessuna lezione (laboratorio a disposizione previo accordo coi tutor per problemi di predisposizione)
- **Mercoledì 25 e venerdì 27 marzo** — lauree
- **Venerdì 3 aprile** — festività pasquali
- **Venerdì 1 maggio** — festa del lavoro

> Restano ore in eccesso: molto probabile salto delle ultime lezioni in aula, ma passibili di cambiamento per necessità di recuperi o altro.
> **LEGGERE L'E-MAIL ISTITUZIONALE NON È UN OPTIONAL!**

---

## 13. Materiale Didattico

- **Libri di testo suggeriti:** elencati sulla guida web del corso. Non seguiti passo passo, ma consigliati per consolidare e approfondire.
- **Si può studiare solo sulle slide? No**, per due motivi: le slide servono solo per tenere il filo del discorso, e metà del corso è pratico. Tuttavia, frequentando regolarmente, prendendo appunti, ripassando subito e facendo domande in aula o a ricevimento, la cosa può funzionare.
- **Risorse aggiuntive:** link su Virtuale, e accesso ai libri tramite la biblioteca d'Ateneo su [ProQuest Ebook Central](https://ebookcentral.proquest.com/lib/unibo/home.action), accessibile anche da fuori rete UniBo via [EZProxy](https://sba.unibo.it/it/almare/servizi-e-strumenti-almare/ezproxy).

### Altro materiale

- **Guida web:** descrizione del corso, regole, contatti
- **Virtuale:** slide proiettate a lezione, tracce delle esercitazioni, link a risorse utili tra cui l'immagine di una **macchina virtuale per VirtualBox** (riferimento per tutti gli esercizi)

---

## 14. Supporto

### Tutor ufficiali

- **Leonardo Barone** — [pagina UniBO](https://www.unibo.it/sitoweb/leonardo.barone4)
- **Pierluca Pevere** — [pagina UniBO](https://www.unibo.it/sitoweb/pierluca.pevere2)

> È stato costituito un **"pool"** coi tutor del corso di Amministrazione di Sistemi. Non contattare il docente o i singoli tutor, ma scrivere alla lista condivisa:
> **AmmSistemi-LabSicurezza@live.unibo.it**

### Contatti del Docente

- **E-mail/Teams:** marco.prandini@unibo.it
- **Telefono:** 051 20 93867
- **Ufficio:** Primo piano del blocco nuovo, in fondo al corridoio oltre l'aula 5.7
- **Ricevimento:** non è stato fissato un orario fisso; benvenuti in ufficio anche senza preavviso, ma consigliato concordare via e-mail.

---

## 15. Modalità d'Esame

### Struttura — Prova unica in due parti (stessa seduta)

| Componente | Peso | Dettagli |
|---|---|---|
| **Prova teorica** | **40%** | Quiz senza materiale, correzione automatica su EOL, risultato immediato. Voto sufficiente richiesto per accedere alla parte pratica. |
| **Prova pratica** | **60%** | Esercizi simili al laboratorio (offensive + defensive), correzione manuale non immediata. Consentito uso di appunti, script autoprodotti, materiale del corso, documentazione. |

**Valutazione finale:** voto in trentesimi, media pesata delle due parti. Entrambe devono risultare singolarmente sufficienti.

### Prova Teorica — Parametri indicativi

- **Durata:** 45 minuti
- **Domande:** 30-40 (vero/falso o scelta multipla, anche su comandi visti in lab)
- **Penalizzazione** per risposta sbagliata
- **Risultato perfetto:** 36 punti
- **Sufficienza:** 18 punti

### Prova Pratica — Temi

- Strumenti di enumerazione, brute-forcing e intercettazione dati
- Sfruttamento di vulnerabilità via privilege escalation, su binari, su applicazioni web
- Sicurezza delle informazioni:
  - Crittografia *(es. consegna file firmati, decifrazione pacchetti da analizzare)*
  - Protezione del traffico con SSH e VPN *(es. accesso a sistema da testare)*
- Configurazione firewall con **nftables**
- Analisi di tracciati di traffico misto lecito/malevolo
- Definizione di regole di monitoraggio e rilevazione intrusioni host- e network-based

### Workflow dell'esame

1. Prova teorica → correzione automatica EOL
2. Se superata → prova pratica
3. Correzione manuale → pubblicazione voti su **AlmaEsami** (orientativamente 7-10 giorni)
4. Entro **4 giorni** dalla pubblicazione: possibilità di rifiutare il voto o richiedere **prova orale integrativa**
   - L'orale può solo modificare un voto già sufficiente (non recuperare un'insufficienza)
   - Può alzare o abbassare il voto **al più di 3 punti**
   - Può spaziare su qualsiasi argomento del corso
5. Scaduto il termine → registrazione automatica dei voti

> **Importante:** Chi ottiene un voto insufficiente alla prova pratica è tenuto a concordare un **ricevimento per visionarla**, e non potrà sostenere altri appelli prima di aver soddisfatto questo requisito.

> **Nota per il corso integrato:** Chi ha in carriera il corso integrato con Amministrazione di Sistemi otterrà come voto la media dei due; è suggerito contattare il docente via e-mail per chiedere la verbalizzazione dopo aver superato entrambi.

---

## 16. Appelli d'Esame

| Periodo | Finalizzato a |
|---|---|
| **Giugno** (2 appelli) | Laurea luglio *(NB: il primo appello potrebbe non avere capienza per tutti; priorità ai laureandi)* |
| **Luglio** | Laurea ottobre |
| **Settembre** | Laurea ottobre |
| **Fine ottobre** (da fissare) | Laurea dicembre |
| **Gennaio 2027** | Laurea febbraio 2027 |
| **Febbraio 2027** | Laurea marzo 2027 |

> Tutti gli appelli sono aperti a tutti gli studenti, ma collocati in modo ottimale per le relative scadenze di laurea → **non necessari (e quindi non concessi) appelli straordinari**. Possibili modifiche per lavori programmati sul plesso.
> **LEGGERE L'E-MAIL ISTITUZIONALE NON È UN OPTIONAL!**
