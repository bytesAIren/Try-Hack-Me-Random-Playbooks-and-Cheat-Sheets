# 🗺️ AI Reconnaissance Methodology, Frameworks & Defense Playbook

- **Dominio:** AI Infrastructure Reconnaissance, Threat Hunting, Attack Surface Management, Compliance & Governance
- **Standard di Riferimento:** MITRE ATLAS, MITRE ATT&CK, OWASP Top 10 for LLMs (2025), NIST AI RMF 1.0, NIST CSF 2.0
- **Scopo:** Guida metodologica end-to-end per condurre ricognizioni su infrastrutture AI/ML, comunicare i risultati tramite framework standard di settore e implementare contromisure di difesa e rilevamento nei SIEM.

---

## 1. Metodologia di Ricognizione AI in 5 Fasi

La procedura segue un approccio strutturato sequenziale, dalla raccolta di intelligence open-source fino all'ispezione della supply chain.

```
[Fase 1: Ricognizione Passiva] ──> [Fase 2: Scansione Attiva] ──> [Fase 3: Fingerprinting API] ──> [Fase 4: Estrazione Metadati] ──> [Fase 5: Revisione Supply Chain]
```

### Fase 1: Ricognizione Passiva (Passive Reconnaissance)
Prima di interagire attivamente con le reti bersaglio, si analizzano le informazioni accessibili pubblicamente:
* **Search Engine per Dispositivi (Shodan / Censys / FOFA):**
  * `port:5000 "MLflow"`
  * `port:8888 title:"Home Page - Select or create a notebook"`
  * `http.title:"Ray Dashboard"`
* **Dorking su Repository di Codice (GitHub):**
  * `filename:.env MLFLOW_TRACKING_URI`
  * `filename:.env HF_TOKEN`
  * `filename:config.json model_name site:github.com`
* **Letteratura e Blog Tecnici:** Ispezione di paper su arXiv o blog post ingegneristici dell'organizzazione per identificare modelli, framework usati e architetture.
* **Container Registries & Recruiting:** Ispezione di DockerHub/GHCR e annunci di lavoro per identificare ruoli specifici (es. *"Kubeflow Platform Engineer"*).

### Fase 2: Scansione Attiva (Active Scanning)
Mappatura dell'infrastruttura di rete con scanner focalizzati sull'AI stack:
* Scansione delle porte caratteristiche (`5000`, `6333`, `8000-8002`, `8080-8082`, `8265`, `8500-8501`, `8888`, `9000-9001`, `11434`, `19530`).
* Individuazione di endpoint gRPC binari (`8001`, `8500`) da approfondire tramite `grpcurl`.
* Rilevamento di metriche Prometheus non protette (`/metrics`).

### Fase 3: Fingerprinting delle API (API Fingerprinting)
Identificazione precisa di engine, librerie e standard di comunicazione:
* Fuzzing delle directory con wordlist mirate alle rotte computazionali AI (`/predict`, `/invocations`, `/v2/models`, `/collections`).
* Analisi degli header HTTP (`Server: TorchServe`, `NV-Status`, `Server: uvicorn`, `x-request-id`).
* Analisi delle risposte a payload malformati (stringhe di debug esclusive come `tensorinfo_map` in TF Serving o namespace `mlflow.tracking`).

### Fase 4: Estrazione dei Metadati (Metadata Extraction)
Mappatura profonda delle risorse una volta confermati i servizi:
* **MLflow:** Elenco esperimenti, modelli registrati, versioni, commit hash nei tag e percorsi dei bucket cloud (`s3://`).
* **Inference Engines (Triton / TF Serving):** Struttura formale dei tensori di input/output, tipi di dati e backend di calcolo.
* **Vector Databases:** Collezioni attive, dimensionalità vettoriale e volumi documentali indicizzati.
* **Jupyter Notebooks:** Kernel attivi ed estrazione di credenziali lasciate in chiaro nelle celle di codice.

### Fase 5: Revisione della Supply Chain (Supply Chain Review)
* Analisi delle fonti esterne dei modelli (Hugging Face Hub, PyTorch Hub).
* Verifica dei permessi sui bucket storage interni contenenti i pesi serializzati (`.pkl`, `.onnx`, `.pt`).
* Audit dei file `requirements.txt` per identificare librerie interne soggette a *Dependency Confusion* su PyPI.

---

## 2. Tabella degli Strumenti per Fase

| Strumento | Funzione Principale | Fase di Riferimento |
| :--- | :--- | :--- |
| **Shodan / Censys / FOFA** | Ricerca globale su banner di servizi AI esposti su Internet. | **Fase 1** (Passiva) |
| **GitHub Search / Dorks** | Individuazione di token (`HF_TOKEN`), credenziali e configurazioni. | **Fase 1** (Passiva) |
| **Nmap (con script NSE)** | Discovery delle porte e rilevamento delle versioni dei servizi. | **Fase 2** (Attiva) |
| **grpcurl** | Interazione con interfacce gRPC ed estrazione schemi Protobuf via Reflection. | **Fase 2** (Attiva) |
| **ffuf / feroxbuster** | Fuzzing delle rotte con dizionari dedicati all'AI stack. | **Fase 2, 3** (Scansione & Fingerprinting) |
| **curl** | Interrogazione HTTP manuale, analisi degli header e test con payload parlanti. | **Fase 3, 4** (Fingerprinting & Metadati) |
| **MLOKit (IBM X-Force)** | Automazione dell'enumerazione ed esfiltrazione da server MLflow. | **Fase 4** (Estrazione Metadati) |
| **Nuclei** | Scansione automatizzata tramite template per vulnerabilità note in servizi AI. | **Fase 2, 3** (Scansione & Fingerprinting) |
| **Agrus Scanner** | Scansione mirata per Shadow AI con probe su oltre 65.535 porte. | **Fase 2** (Attiva) |

---

## 3. Mappatura sui Framework di Settore (Industry Frameworks)

La mappatura delle tecniche e delle evidenze di ricognizione sui framework internazionali consente di comunicare i risultati a security leader, auditor e compliance team in un linguaggio condiviso e standardizzato.

### A. MITRE ATLAS (Adversarial Threat Landscape for AI Systems)
ATLAS è il framework principale dedicato specificamente alle minacce e alle tattiche avversarie contro i sistemi di Machine Learning:

| Attività / Riscontro nella Ricognizione | ATLAS Technique ID | Nome della Tecnica |
| :--- | :--- | :--- |
| Dorking Shodan e GitHub per infrastrutture AI | **AML.T0000** | Active Scanning |
| Individuazione di Model Registry e artifact tramite API non protette | **AML.T0048** | Discover ML Artifacts |
| Rilevamento di token HF esposti e dependency confusion | **AML.T0040** | ML Supply Chain Compromise |
| Enumerazione di configurazioni LLM e compatibilità degli schemi API | **AML.T0069** | Discover LLM System Information |
| **Insieme completo delle attività di ricognizione sull'AI** | **AML.TA0002** | Reconnaissance (Tactic) |

### B. MITRE ATT&CK (Enterprise)
Poiché la ricognizione su sistemi AI condivide tecniche consolidate con la ricognizione di rete convenzionale, si applicano anche le tattiche tradizionali:

| Attività / Riscontro nella Ricognizione | ATT&CK Technique ID | Nome della Tecnica |
| :--- | :--- | :--- |
| Scansione porte per servizi specifici dell'AI stack | **T1046** | Network Service Scanning |
| Estrazione della topologia di deployment da metriche e metadati | **T1592** | Gather Victim Host Information |
| Probing per interfacce di gestione non autenticate | **T1595.002** | Vulnerability Scanning |
| Raccolta preventiva di intelligence sull'infrastruttura AI | **TA0043** | Reconnaissance (Tactic) |

### C. OWASP Top 10 for LLM Applications (2025)
Le evidenze emerse dalla ricognizione si collegano direttamente alle categorie di rischio OWASP per i modelli linguistici:

| Riscontro nella Ricognizione | OWASP LLM ID | Nome del Rischio & Spiegazione |
| :--- | :--- | :--- |
| Server MLflow esposti, notebook Jupyter e API prive di autenticazione | **LLM05** | **Improper Output Handling:** Perdita di informazioni e metadata sensibili attraverso le risposte delle API. |
| Artifact di modello scaricabili da registri non protetti | **LLM06** | **Excessive Agency:** Accesso diretto ai modelli senza controlli autorizzativi a monte. |
| Token HF esposti, dependency confusion, modelli avvelenati da hub pubblici | **LLM03** | **Training Data Poisoning / Supply Chain Vulnerabilities:** Compromissione della fiducia nei componenti esterni. |
| Credenziali di default e assenza di autenticazione su MLflow/Kubeflow | **LLM10** | **Model Theft:** La ricognizione abilita l'esfiltrazione diretta degli interi file binari proprietari. |

### D. NIST AI Risk Management Framework (AI RMF 1.0)
Il framework NIST AI RMF struttura il rischio in quattro funzioni (**Govern, Map, Measure, Manage**). Le attività di ricognizione supportano principalmente la funzione **Map**:

* **Map 1.1:** *I componenti dei sistemi AI e le loro interazioni sono identificati.* È l'obiettivo primario della ricognizione: non è possibile gestire il rischio di componenti non censiti.
* **Map 1.5:** *I rischi potenziali del sistema AI sono valutati.* La mappatura della superficie d'attacco (porte esposte, registri privi di credenziali) costituisce la valutazione del rischio a livello infrastrutturale.
* **Map 3.2:** *I rischi legati a risorse AI di terze parti sono identificati.* L'ispezione della supply chain (token Hugging Face, dipendenze PyTorch Hub, registry pubblici) ricade interamente in questo controllo.
* **Measure 2.6:** *Esistono processi per determinare se i sistemi AI funzionano come previsto.* L'enumerazione delle metriche Prometheus e delle interfacce di debug supporta questa funzione, poiché la presenza di endpoint aperti indica un sistema non conforme dal punto di vista della sicurezza.

### E. NIST Cybersecurity Framework (CSF 2.0)
Le attività si allineano con la funzione fondamentale **Identify (ID)** del CSF 2.0:

* **ID.AM (Asset Management):** Identificazione e inventario di tutti i componenti infrastrutturali dell'AI (server d'inferenza, cluster di training, vector DB).
* **ID.RA (Risk Assessment):** Mappatura della superficie di attacco e identificazione delle misconfigurazioni che introducono vulnerabilità nel perimetro aziendale.

---

## 4. Prospettiva Difensiva: Rilevamento nei Log e nel SIEM

Ogni fase di ricognizione lascia impronte evidenti nei log applicativi, di rete e nei controlli di telemetria.

### Pattern Riconoscibili nel SIEM
1. **Scansione Sequenziale dei Modelli:** Raffiche di richieste `GET` verso `/v2/models` da un unico indirizzo IP esterno in brevi finestre temporali.
2. **Accesso Scriptato alle API di MLflow:** Chiamate dirette a `/registered-models/list` prive dei normali cookie di sessione della UI web (tipico di tool come MLOKit).
3. **Scraping Anomalo di Prometheus:** Richieste HTTP verso `/metrics` (porte `8002`, `8082`) provenienti da indirizzi IP esterni alla subnet del monitoraggio autorizzato.
4. **Port Scanning Specialistico:** Connessioni ravvicinate sulle porte esclusive `5000`, `6333`, `8000`, `8081`, `8265`, `8500`, `8888`.
5. **Tentativi di Path Traversal:** Sequenze `../` o `%2e%2e%2f` negli URI degli artifact di MLflow (probe per CVE-2024-1558 o CVE-2026-2033).
6. **Accesso a Jupyter senza Token:** Richieste non autenticate agli endpoint `/api/kernels` e `/api/contents`.
7. **TLS Fingerprinting (JA3 / JA4):** Le scansioni automatizzate basate su librerie Python (`requests`, client gRPC) presentano firme JA4 distinte rispetto ai browser tradizionali. In campagne reali di ricognizione, oltre il 99% delle sonde condivide la medesima firma JA4H anche se distribuita su decine di IP diversi.

---

## 5. Quick Wins: Mitigazione & Riduzione della Superficie d'Attacco

Contromisure immediate per blindare l'infrastruttura AI:

1. **Autenticazione Obbligatoria su MLflow:** Configurare `MLFLOW_TRACKING_USERNAME` e `MLFLOW_TRACKING_PASSWORD` o proteggere l'istanza con un Reverse Proxy autenticato.
2. **Hardening di Jupyter:** Disabilitare sempre i flag `--allow-root` e `--ip=0.0.0.0` in produzione. Imporre token crittografici e confinare i notebook dietro VPN o ingress controller.
3. **Segregazione delle Porte a Livello Firewall:** Bloccare qualsiasi accesso diretto da Internet a `5000`, `6333`, `8000-8002`, `8080-8082`, `8265`, `8500-8501`, `8888`, `9000`.
4. **Blocco dei Modelli Dinamici su Triton:** Avviare il server con `--model-control-mode none` per impedire il caricamento/scaricamento dinamico non autorizzato dei modelli.
5. **Isolamento delle Metriche:** Restringere l'accesso a `/metrics` unicamente all'intervallo IP del server Prometheus aziendale.
6. **Token Hugging Face a Minimo Privilegio:** Adottare esclusivamente token a grana fine (*fine-grained tokens*) in sola lettura e con scadenza temporale.
7. **Sanitizzazione di Errori ed Header:** Disabilitare la modalità di debug in produzione ed eliminare gli header di telemetria hardware (`NV-Status`, GPU metrics) prima dell'esposizione a reti non fidate.
8. **Audit delle Policy sui Bucket di Storage:** Verificare che i bucket S3/GCS contenenti gli artifact dei modelli rifiutino l'accesso anonimo in lettura o scrittura.

---

## 6. Case Study: Compromissione di Hugging Face Spaces (Giugno 2024)

* **Incidente:** Accesso non autorizzato a container di sviluppatori su Hugging Face Spaces con estrazione di segreti e token HF.
* **Lezione:** Ha dimostrato l'impatto critico della **Fase 1** (Ricognizione Passiva) e della **Fase 5** (Supply Chain). La presenza di token con privilegi di scrittura in ambienti applicativi condivisi o configurazioni `.env` espone l'intero patrimonio intellettuale e i modelli proprietari dell'organizzazione.
