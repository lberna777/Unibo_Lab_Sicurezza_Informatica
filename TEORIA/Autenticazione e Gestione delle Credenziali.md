# **Laboratorio di Sicurezza Informatica — Autenticazione**

**Docente:** Prof. Marco Prandini  
**Argomento:** La regola AAA, gestione sicura delle password (Hash, Salt, Pepper), Autenticazione Attiva, 2FA/MFA e Standard FIDO.

## **1\. La Regola AAA**

La gestione degli accessi a un sistema si fonda su tre pilastri fondamentali, noti come regola AAA:

* **Autenticazione (Authentication):** Attribuzione certa dell'identità di un soggetto che utilizza le risorse. Spesso include un'identificazione preliminare (es. inserimento dell'username). *Errore comune: usare elementi identificativi pubblici come se fossero "segreti" per autenticare.*  
* **Autorizzazione (Authorization):** Verifica dei diritti di un soggetto di compiere una determinata azione su un oggetto (concessione o negazione del permesso).  
* **Auditing (Accounting):** Tracciamento affidabile di tutte le decisioni di autenticazione e autorizzazione. Serve a verificare l'efficacia delle policy (trovando un compromesso tra utilità e usabilità).

## **2\. I Fattori di Autenticazione**

L'autenticazione si basa sulla dimostrazione da parte di un utente (Prover) a un sistema (Verifier) di possedere un elemento identificativo. I fattori si dividono in:

1. **Conoscenza (Qualcosa che sai):** Password, PIN, risposte a domande di sicurezza.  
2. **Possesso (Qualcosa che hai):** Smart card, token hardware (es. YubiKey), smartphone.  
3. **Inerenza / Biometria (Qualcosa che sei):** Impronta digitale, scansione dell'iride, riconoscimento facciale.  
4. **Posizione (Dove sei):** Geolocalizzazione GPS, accesso da una specifica rete aziendale.

## **3\. Autenticazione Passiva e Protezione delle Password**

Nell'autenticazione passiva, il Prover invia il segreto (la password) al Verifier per dimostrare di conoscerlo.  
Questo approccio presenta gravi problemi:

* **Invio in chiaro:** Vulnerabile all'intercettazione (Sniffing).  
* **Invio offuscato/cifrato in modo statico:** Vulnerabile ai *Replay Attack* (l'attaccante cattura il pacchetto e lo reinvia identico).  
* **Memorizzazione:** Se il Verifier subisce un furto del database, le password vengono compromesse.

### **Memorizzazione Sicura: Hash, Salt e Pepper**

Per proteggere il database del Verifier, le password **non devono mai essere salvate in chiaro**.

1. **Hashing semplice:** Si applica una funzione crittografica unidirezionale (Hash) alla password. Il Verifier salva solo l'impronta (fingerprint). Quando l'utente fa login, il sistema calcola l'hash della password immessa e lo confronta con quello salvato.  
   * *Problema:* Vulnerabile agli attacchi a dizionario, Rainbow Tables e non protegge utenti con la stessa password.  
2. **Salt:** È una stringa casuale (generata da un PRNG) generata al momento della creazione della password e concatenata ad essa prima dell'hashing (Hash(Password \+ Salt)). Il Salt viene salvato in chiaro nel database accanto all'hash.  
   * *Vantaggio:* Due utenti con la stessa password avranno hash completamente diversi. Rende inutili le tabelle precalcolate (Rainbow Tables).  
   * *Esempio Pratico (File /etc/shadow Linux visto in laboratorio):* $6$ViDM2ltuaSN...$lp40Uo... \-\> Il "6" indica l'algoritmo (SHA-512), seguito dal *Salt*, e infine dall'*Hash*.  
3. **Pepper:** Aggiunge un ulteriore livello di sicurezza. È un segreto (una chiave crittografica) che viene unito alla password, ma a differenza del Salt, **non** viene salvato nel database, bensì in un modulo hardware sicuro separato (HSM \- Hardware Security Module).

**Nota di Laboratorio:** Il Salt è inefficace contro gli attacchi offline (come quelli eseguiti con *John The Ripper*) se la password scelta è debole. L'attaccante, possedendo il file /etc/shadow, ha già il Salt e può semplicemente ricalcolare l'hash per ogni parola del dizionario.

## **4\. Usabilità e Cattive Pratiche (Password Policies)**

L'efficacia di una password dipende dalla sua entropia (imprevedibilità) e dalla sua memorizzabilità.  
Come illustrato dal celebre fumetto *XKCD 936*:

* Una password come Tr0ub4dor&3 è **difficile da ricordare ma facile da indovinare** per un computer (circa 3 giorni di calcolo).  
* Una *passphrase* come correct horse battery staple (quattro parole comuni casuali) è **facile da ricordare e difficilissima da indovinare** (550 anni di calcolo).

**Cattive pratiche comuni nei portali web:**

* Troncamento silenzioso della password (es. vengono considerati solo i primi 8 caratteri).  
* Limitazioni ingiustificate sui caratteri speciali consentiti.  
* *Information Leakage:* Messaggi di errore come "Nome utente già esistente" in fase di registrazione, che facilitano l'enumerazione degli utenti validi (OSINT).

La soluzione moderna raccomandata è l'utilizzo di **Password Manager** protetti da un'unica passphrase molto robusta e da MFA.

## **5\. Autenticazione Attiva**

Nell'autenticazione attiva, il Prover convince il Verifier di possedere il segreto *senza mai svelarlo* e inviando ogni volta un dato diverso. Questo neutralizza il furto del database e i Replay Attack.

* **S-KEY (One-Time Password tramite Hash Chain):**  
  * Il sistema genera una catena di hash applicando la funzione ![][image1] volte a un segreto ![][image2]: ![][image3].  
  * Ad ogni login, l'utente invia il valore dell'hash precedente ![][image4].  
  * Il Verifier calcola un solo hash sul dato ricevuto e verifica se combacia con quello salvato (![][image5]). Se corretto, il nuovo valore diventa il riferimento per il login successivo.  
* **Sistemi Challenge-Response (Crittografia Asimmetrica):**  
  * Il Verifier invia una sfida casuale (*Nonce*).  
  * Il Prover la firma/cifra usando la propria Chiave Privata.  
  * Il Verifier controlla la risposta usando la Chiave Pubblica del Prover.

## **6\. MFA, 2FA e 2SA**

Le credenziali singole sono facilmente rubate tramite Phishing o Data Breach (es. account compromessi verificabili su *HaveIBeenPwned*). L'aggiunta di fattori mitiga questo rischio.  
È fondamentale distinguere i termini:

* **MFA (Multi-Factor Authentication):** Utilizzo di più di due fattori per l'autenticazione.  
* **2FA (Two-Factor Authentication):** Utilizzo di esattamente **DUE fattori DISTINTI** (es. Password \[Conoscenza\] \+ Token Hardware \[Possesso\]).  
* **2SA (Two-Step Authentication/Verification):** Richiede due passaggi, ma **non necessariamente due fattori distinti**. Ad esempio, Password \+ SMS con codice OTP. *L'SMS o l'Email non sono considerati veri fattori di "Possesso" perché facilmente intercettabili tramite attacchi MITM, SIM Swapping o compromissione dell'account email.*

**Tipologie di Token Temporanei:**

* **OTP (One Time Password):** Inviato via SMS o Email. Valido per un singolo utilizzo.  
* **TOTP (Time-based One Time Password):** Generato da app come Google Authenticator. Valido per un singolo utilizzo in una finestra temporale ristretta (es. 30 secondi).

## **7\. Standard FIDO (Fast IDentity Online)**

La FIDO Alliance (supportata da Google, Microsoft, ecc.) sviluppa standard aperti per un'autenticazione più sicura e usabile, basata su crittografia a chiave pubblica.

* **FIDO UAF (Universal Authentication Framework):**  
  * Progettato per il **login Passwordless** (senza password).  
  * L'utente usa il proprio dispositivo (es. smartphone) tramite un fattore locale (es. impronta digitale) per sbloccare una chiave privata memorizzata nel dispositivo (che non lascia mai l'hardware). Il dispositivo firma una *challenge* del server.  
* **FIDO U2F (Universal Second Factor) / FIDO2:**  
  * Semplifica e rafforza la 2FA usando chiavi USB, NFC o Bluetooth (es. **YubiKey**).  
  * Fornisce una forte protezione nativa contro Phishing, Man-In-The-Middle e dirottamento di sessione.

## **8\. Vulnerabilità Hardware: Il caso WebUSB e YubiKey**

Nessun sistema è totalmente inviolabile.  
Una funzionalità chiave della YubiKey (U2F) è l'**Origin-Check**: la chiavetta verifica l'effettivo dominio del sito per evitare che l'utente inserisca il token su un sito di phishing.

* **L'attacco (Vervier e Orrù, 2018):** In Chrome 61 fu introdotta la feature **WebUSB**, che permetteva a un sito web di dialogare direttamente con dispositivi USB tramite JavaScript.  
* I ricercatori hanno dimostrato che una pagina malevola poteva usare WebUSB per inviare richieste U2F direttamente all'interfaccia CCID della YubiKey NEO, bypassando i controlli del browser e **ingannando la chiavetta sull'origine reale del sito web**.

[image1]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABIAAAAYCAYAAAD3Va0xAAABG0lEQVR4Xu2TvS5EURRGPyREIhKvIEFBIzQKCQ2FXrwA0UxFpVN5A1FQyESmFB2JJ9CTjBk/EcV0EsZPxzrZZ8y5e4wZtVnJys399j475957rtThr8xiCe/xLl6LuFhv0XbMy3iDe0mtgRP8xAVfgHF8wUOcx65sOUsF37DX5WHhGQ67/EdGZbs5d/kG7mO/y5uyKhu0Fe8HMI/r3x1tciQbNINjeIkXmY42eZS9zGU8xmvZ4Km0qRUjskUfuIM9mIvZQdLXkjXZonBWagzKdhi+4lCS/0pBNmja5bsx33R5U8L5eZY9UsqEbFA47d2u1sCkrPnUFyJXsvqKL9SYk/1fT/iOr/iAS7Heh7dYle02vKvwn/ldd/i/fAGarzw7MxKyOQAAAABJRU5ErkJggg==>

[image2]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABIAAAAYCAYAAAD3Va0xAAABF0lEQVR4Xu2TvUoDQRRGr1qZSkTSpBQMIjZ5AAvjA0jE0jYJ1jZ2go1VQLCxsgkk8SUELRQUkQTSBEJIo502kkLUnHFm2J27m8J+Dxx25n7D/C0jkvFftnGIr/iGzTD+4xFHOBA7thGkijZ+4A+uqmwBT/AWC2GUpItH+CvpK57hvi5qiniNS/iJ75gLRojcYV7VEtTw0LUvxe6qGsWyiM+x/kxauO7am2InMkf1lPEi1p/Ji+rfiJ1sy/VPcS+K0/H3E6cidiJfN/ezEsXp1CW6H4/53WP8wjV8CuN0Orihi3Asdlf3eK6yBHPYd1+NOcpE7GS7KktwgD2c14HjCr9xWQeeHbHvyqxoNE/DvDlNCR90MSMDpvMbNCf6RtASAAAAAElFTkSuQmCC>

[image3]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADQAAAAYCAYAAAC1Ft6mAAADMklEQVR4Xu2XaahNURTHl3km85QMmX1QfCA+UHyhFEIoiTIlQxFCUuKDKIQoZJYQUYZkTCIUMkQoY2Yyz/z/b5193z7r7Hvee70rvfKrX/fetc60z9lr7XNF/iOT4D041CZKKqXgK9jKJrJQE1aywWLA41W2wTSqwSo26NERPrHBLDSC52ENmygG7eEZSb/GPBbDr/A3nG5yPlPhjuj7CDgPls5PZ6gAz8FRNhGxDz6CT+Fj2DmelmHwIbwL78DbsGWUWwr3S/i8McaKDqirTXjwQibCybAnfC86DSxL4AXRKZqNtvCt6Dk3mRypB1+K1mt5L14dPoNjvFiQtaIXWNYmIlz9nIbNRLfjRVl4wnewm00YxsNp8AX8AuvG03nT6qqJOUbDBxIfaIJb8LANerj6GQivw9bxdIYp8JoNBuDUZU0sEn1Ks+Np6Q1XmZijDPwGB9iEo6HoQWeKPuo+sGlsC23Z26PvHDgHNgOWy2yhHBWdmgVxOfpsAn+I3nFeqGMBHOT9trCuVtqgg/OUA9oJd8EJosW40NtmMxwZfefA10t4PeJ+rKE02oiey7Fb9Py8SY5TkpyGPgfhFRt0rBE94BwvxsG8lvxuwpbuY38TNxVYH2mME71pjh6i5z8e/eZacyk/HWSFaP0FuSna3302iA6oKHCa8sJ62YTB1Y8PGwD37SBaP7zgNNhpf0qgfdeX5NMh9+EeEyuITqLH4mcaoanCNsx9V4vODn/6heA6+EsCnW6I6IH8Ntslig33YoWhueh+acXcTrROLZxmnBEfRAdcO55OwEWdi3ICdgoexO9Wy+FnWBUOhv29XBqsKw5olk14cGFmxwzBZsL9XQdMg03KlkkenLtHTIyvLcdEF9MTUoh3Jw++0qyzwYha8KLoTQrRQrQultlEgLOig4rB+fddkq8RvIN87BxUQQVu2QhPmhg5AD/CT6JP/1A8nWEv7GuDAZ6LroMJGthARB3R15iiwvphO61oEzmEizFfprNde05hG70hya6ZS7bBrTb4N+kn+qLb2CZyQHf4Rgr/JzNnzIVbbLCY8OnzbwnfNf8J80VrMVfwvxr/OvynRPIHU1GeBv0kDo4AAAAASUVORK5CYII=>

[image4]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAEYAAAAYCAYAAABHqosDAAADhklEQVR4Xu2YaahNURTHlzFzyEx5kTmRDwjphJSizEnki1nkg0hkyJAUIooPxmRMyQehpFCEZJbMylAyZQoZ/v+3zunus96555z33n3Xe/V+9evds9a+5567795r7/1EKskZc+ETOMEmiklzON0GKzJV4DvYwSaKwRZ4Cr60iRha2UApaW0DSdSHdW3QoQd8ZYMlwJP0HbMULrTBUrJXUo769fAH/AsXmJzLfHjQfz0ZLoNVM+nUeJKuY0bBB7CmTYBBotP6NXwD94fThVyFz+BD0bYb/XhL+BYO8K9j4Zxnx/S1CYfjcA6cJ/rlPsNGboOUeJLcMXx43n+ETRgOwY/wD2xvctXgCnheik6fRfAprGHiRdgh+iDVbcInqC/8kALRdp2dPDt2SRZnO+2IJ8lTcqVoLUrilugo548ajAiXdXCcDYqOQo6kmTZh4ZCNe5CgvoyGd2HHcLrwg2pl0U4FT3QKZIPtOT3G2oShEzwCG8Kv8AOsE2ohcgE2M7EAloKbNujCYcse5/DiTYbBtqEWulQf8F+zA9lBLIqJQzECT+I7pr/o8/S0CcMMyYxGjni+x90G1IbXnWvLRNEp2NQmAlihedPD8CicBR/BNU6bfXCK/5oduFNSVnYDp8hJ+AnugcNDWWWS6PPUswkDF4Iu/uvuou/h1AoYDLc615beou9hkY9ku2gD1oMAdsp7yaw6XMpd7HUu4RDnVErihrk+J/o9BvrXq+CYTLoIjUXbc0GJ5D68aGK7RDvmf7BbtDbEEdQXF05vftEgzns0yaQj4XdcboOE23M7WshzeMzE8sUJ3zi4mtjVjsvzC/hLdHG4Fk5H8hiutkEyXrRj+jmxPn6Mxel/wJ3pHRs0sB52s0GwWPTZL8HNJmdhR/6E02yCsDh9kfDqwht+Fy1+3AOMdHL5YAP8Jrp3ioLxe/5fC6cOn52dk/Tc7UTbDbEJwip+2sQuw7OiH8yCFnd+Kgumij5wG5vw4XHktmQ/jrBG/RYtrnEMFf0cdlAIbqQ4H+1Q4p6FGx92Dpe8fFMg+sBeOFz4y3K14oigPArwzGTpJfrjJsEaxW1D5G6/hQ34cEg2sME8whoTd6DNBdyLJdWhcgc3jzwBc7tfFnQV/Y+CPdqUe1g/rsBNNpEjzsBtNlhR4BmO+wz3FJ8LuFrxvwQlOeeVG3gWyrplLyFrJXlHXEklKfkHHFqujuDZZF4AAAAASUVORK5CYII=>

[image5]: <data:image/png;base64,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>