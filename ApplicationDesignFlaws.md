# 🛡️ Web Vulnerabilities & API Cheat Sheet (OWASP Top 10 2025)

```
[ Error Leaks ] ➔ [ Supply Chain Backdoors ] ➔ [ Hardcoded Crypto ] ➔ [ Insecure Design / Hidden APIs ]
```

---

## 1. Verbose Errors & Debug Leaks (Security Misconfiguration - A05)

### 📌 Cosa succede sotto il cofano:
L'applicazione gira con la modalità di **Debug attiva** in produzione (es. Flask `debug=True`, PHP `display_errors=On`). Quando riceve un dato non previsto, va in crash e stampa a schermo il **Traceback**: file del server, righe di codice sorgente, variabili d'ambiente e segreti.

### 🔍 Il campanello d'allarme:
L'interfaccia dice frasi categoriche tipo: *"User ID must be numeric"*, *"Enter valid email"*.

### 🛠️ Come testare (Negative Testing / Fuzzing di base):
Non dare mai all'endpoint il formato che si aspetta:
```bash
# 1. Se vuole numeri, passa lettere o caratteri speciali
curl http://target:port/api/user/abc
curl http://target:port/api/user/'

# 2. Passa valori limite o nulli
curl http://target:port/api/user/0
curl http://target:port/api/user/-1
curl http://target:port/api/user/null
```
* **Cosa cercare nella risposta:** percorsi di file (`/app/app.py`), nomi di moduli (`traceback`), token o variabili segrete passate nei messaggi di errore.
* **🛡️ Rimedio difensivo:** Disattivare il debug in produzione. Intercettare le eccezioni con blocchi `try/catch` e restituire solo codici HTTP generici (es. `400 Bad Request`) con messaggi anonimizzati.

---

## 2. Supply Chain Backdoors & Easter Eggs (Vulnerable Components - A06)

### 📌 Cosa succede sotto il cofano:
L'applicazione importa una libreria o dipendenza esterna (npm, pip, file `.py` inclusi) obsoleta, non verificata o malevola. Lo sviluppatore del sito è innocente, ma la libreria include logiche nascoste, backdoor o modalità di "test/debug" dimenticate.

### 🔍 Il campanello d'allarme:
Il testo della challenge o i file del progetto menzionano librerie vecchie (`vulnerable_utils.py`, librerie `@1.0.0`) o invitano a fare *"debug"*.

### 🛠️ Come testare:
Cerca parametri o parole chiave "magiche" che deviano il normale flusso del codice:
```bash
# 1. Testa parole chiave di stato o debug nel payload JSON
curl -X POST http://target:port/api/process \
  -H "Content-Type: application/json" \
  -d '{"data": "debug"}'

# 2. Testa parole come: admin, test, dev, config, version, help
curl -X POST http://target:port/api/process -d '{"data": "admin"}'
```
* **🛡️ Rimedio difensivo:** Implementare l'analisi delle dipendenze (SCA - Software Composition Analysis) nella pipeline CI/CD (es. `npm audit`, `pip-audit`, Snyk), bloccare versioni obsolete e usare solo pacchetti firmati e verificati.

---

## 3. Hardcoded Keys & Client-Side Crypto (Cryptographic Failures - A02)

### 📌 Cosa succede sotto il cofano:
Il server mostra dati cifrati (spesso in Base64) sostenendo che solo gli amministratori hanno la chiave. In realtà, lo sviluppatore ha inserito la **chiave segreta direttamente dentro uno script JavaScript o un file statico pubblico**.

### 🔍 Il campanello d'allarme:
* Pagine che dicono: *"Document is encrypted for security. Decryption key required"*.
* Blocchi di testo apparentemente incomprensibili ma che finiscono per `=` o `==` (firma di stringhe Base64).

### 🛠️ Come scovare la chiave:
1. **Apri il codice sorgente (Ctrl + U):**
   * Cerca tag `<script src="...">` o file `.js` associati.
2. **Cerca costanti tipiche:**
   * Parole chiave: `SECRET_KEY`, `AES`, `KEY_SIZE`, `MODE`, `IV`, `encrypt`, `decrypt`.
3. **Decifrare con CyberChef:**
   * Se il testo è Base64 $\rightarrow$ operazione **`From Base64`**.
   * Aggiungi l'operazione di decifratura (es. **`AES Decrypt`**).
   * Imposta i parametri trovati nello script:
     * **Key:** solitamente `UTF8` (o `Hex` se in byte).
     * **Mode:** `ECB`, `CBC`, `GCM`.
* **🛡️ Rimedio difensivo:** **Mai fare crittografia con chiavi simmetriche lato client.** I dati confidenziali devono essere decifrati solo dal backend autenticato e inviati all'utente autorizzato tramite connessione protetta (TLS). Le chiavi vanno salvate in gestori dedicati (KMS, HashiCorp Vault, variabili d'ambiente protette).

---

## 4. Insecure Design & Client Assumption (Insecure Design - A04)

### 📌 Cosa succede sotto il cofano:
Gli sviluppatori confondono la **grafica/interfaccia (il frontend)** con la **sicurezza (il backend)**. Assumono che:
* *"Se mettiamo una schermata che dice 'Solo per Mobile', gli utenti desktop non vedranno i dati"*.
* *"Se non mettiamo il pulsante 'Cancella utente' nella pagina HTML, nessuno potrà cancellare utenti"*.
In realtà, le API dietro le quinte sono esposte pubblicamente e prive di controlli di autorizzazione.

### 🔍 Il campanello d'allarme:
Siti o landing page che bloccano l'accesso in base al dispositivo (*"Download our mobile app"*, *"Desktop browser not supported"*), oppure pannelli con pulsanti disabilitati in grigio.

### 🛠️ Come testare la Progettazione Insicura:

#### A. Bypass dell'User-Agent (Spoofing)
Se il controllo è solo sull'intestazione HTTP:
```bash
# Simula un dispositivo iPhone o Android
curl -s http://target:port/ -H "User-Agent: Mozilla/5.0 (iPhone; CPU iPhone OS 16_0 like Mac OS X)..."
```

#### B. Esplorazione diretta delle API (Bypasso totale del Frontend)
Non usare il sito web: parla direttamente con i suoi endpoint nascosti. Le app mobile usano percorsi convenzionali:
```bash
# 1. Cerca elenchi di risorse o utenti
curl -s http://target:port/api/users
curl -s http://target:port/api/profile

# 2. Testa l'accesso diretto a ID o ruoli privilegiati (BOLA / IDOR)
curl -s http://target:port/api/messages/admin
curl -s http://target:port/api/messages/1
curl -s http://target:port/api/admin/messages
```
* **🛡️ Rimedio difensivo:** **Zero Trust.** Non fidarsi mai del client. Ogni richiesta API deve richiedere autenticazione esplicita (token JWT, sessione valida) e autorizzazione rigida (RBAC: *questo utente ha il diritto di leggere questo specifico record?*), a prescindere dal tipo di dispositivo che chiama l'endpoint.
