# ⚠️ AI & MLOps Platform Misconfigurations, Supply Chain Reconnaissance & MITRE ATLAS Mapping

- **Dominio:** AI Infrastructure Reconnaissance, MLOps Attack Surface Mapping, Supply Chain Security
- **Standard di Riferimento:** MITRE ATLAS (AML.T0006: Active Scanning, AML.T0007: Discover ML Artifacts, AML.T0010: ML Supply Chain Compromise, AML.T0014: Discover ML Model Family, AML.TA0002: Reconnaissance)
- **Scopo:** Documentare le misconfigurazioni più diffuse sulle piattaforme ML, le tecniche di ricognizione sulla supply chain dell'IA e la mappatura tassonomica dei rischi.

---

## 1. Misconfigurazioni Diffuse nelle Piattaforme MLOps

Molte piattaforme di Machine Learning presentano impostazioni di default orientate alla facilità d'uso negli ambienti di sviluppo che, se replicate in produzione, diventano vulnerabilità sfruttabili.

### A. MLflow

* **Assenza di Autenticazione:** Nelle versioni precedenti alla 2.x, MLflow veniva distribuito privo di autenticazione abilitata di default.
* **Credenziali di Default Hardcoded (CVE-2026-2635):** Anche a seguito dell'introduzione del modulo di autenticazione, il file `basic_auth.ini` conservava credenziali predefinite in chiaro. Scansioni su larga scala sulla porta `5000` consentivano l'autenticazione tramite queste credenziali note.
* **Directory Traversal & Arbitrary File Read (CVE-2026-2033):** Una vulnerabilità nel gestore degli *artifact* consentiva l'accesso non autenticato al file system sottostante (valutazione CVSS 9.8).

### B. Kubeflow Dashboards

* **Mancanza di Integrazione OIDC:** Le dashboard di Kubeflow vengono frequentemente esposte tramite i servizi Kubernetes `LoadBalancer` o `NodePort` senza autenticazione federata.
* **Escalation sul Cluster:** Un utente non autenticato può accedere liberamente alla dashboard e creare *Jupyter Notebooks*. Questi ultimi vengono spesso associati a *Kubernetes Service Accounts* dotati di permessi elevati (RBAC) a livello di cluster, offrendo un canale diretto dalla dashboard al piano di controllo di Kubernetes.

### C. TorchServe (PyTorch)

* **Management API Esposta (`:8081`):** Consente la registrazione dinamica di nuovi modelli a partire da URL arbitrari.
* **Caricamento di Archivi Malevoli (`.mar`):** Se la porta è accessibile, un utente malevolo può istruire il server a scaricare ed eseguire un pacchetto `.mar` personalizzato. Poiché TorchServe esegue codice di inizializzazione Python durante la fase di caricamento del modello, questo meccanismo può condurre all'esecuzione remota di codice (RCE).

### D. AWS SageMaker Notebooks

* **`DirectInternetAccess: Enabled`:** Abilita la raggiungibilità diretta dei notebook da Internet tramite indirizzo IP pubblico.
* *Incidenza nei report di sicurezza:* Un'ampia percentuale di ambienti organizzativi che utilizzano SageMaker ha presentato almeno un notebook configurato con accesso diretto da rete esterna.

---

## 2. Model Registries: Il Punto Critico dell'Architettura ML

I *Model Registry* (es. MLflow Registry, SageMaker Model Registry) rappresentano uno dei target a più alto valore informativo durante le fasi di audit e ricognizione. Un registro non memorizza esclusivamente i file binari, ma l'intera storiografia di sviluppo:

* Nomi dei modelli e struttura dei progetti
* Cronologia completa delle versioni e stato dei deployment (`Staging`, `Production`, `Archived`)
* Timestamp di creazione e ID degli utenti (Data Scientist e Service Accounts)
* URI degli artifact che collegano direttamente ai bucket di storage cloud aziendali (`s3://`, `gs://`, `azureblob://`)
* Tag e commit hash collegati ai repository del codice sorgente

```
                               ┌──> S3 / Cloud Storage (Artifacts)
[Unsecured Model Registry] ────┼──> Git Commit Hashes & Source Code Paths
                               └──> Contributor User IDs & Environment Metadata
```

---

## 3. Supply Chain Reconnaissance nei Sistemi AI

L'infrastruttura di intelligenza artificiale moderna dipende fortemente da repository esterni, pacchetti terzi e modelli pre-addestrati.

### Exposure di Token Hugging Face

* **Leakage di Credenziali:** L'identificazione di token d'accesso (`HF_TOKEN`) all'interno di file `.env`, log di build CI/CD o risorse Kubernetes consente di accedere ai modelli e dataset privati dell'organizzazione ospitati sulla piattaforma Hugging Face.

### Dependency Confusion nelle Pipeline ML

* **Pacchetti Interni non Registrati:** Le pipeline di addestramento utilizzano file `requirements.txt` articolati con riferimenti a moduli interni (es. `company-data-utils`).
* **Iniezione di Pacchetti Typosquatted:** Se un nome di pacchetto interno non risulta registrato nei repository pubblici ufficiali (es. PyPI), l'eventuale registrazione da parte di terzi dello stesso nome su PyPI può spingere i processi di build automatizzati (come le pipeline Kubeflow) a scaricare la dipendenza esterna in fase di addestramento.

### Mappatura delle Fonti dei Modelli

* Le dipendenze dei modelli pre-addestrati scaricati da *Hugging Face Hub* o *PyTorch Hub* sono esposte all'interno dei file di configurazione e nei log dei container. La compromissione dell'origine del modello o la sostituzione del file remoto impatta direttamente la catena di fiducia dell'intera applicazione a valle.

---

## 4. Mappatura sul Framework MITRE ATLAS

Le fasi di ricognizione e analisi dell'architettura AI si mappano direttamente sulla matrice **MITRE ATLAS** (Adversarial Threat Landscape for Artificial-Intelligence Systems):

| Contenuto / Attività | Tecnica / Tattica MITRE ATLAS | Identificativo ATLAS |
| :--- | :--- | :--- |
| Port scanning per identificare servizi AI/ML ed endpoint esposti | **Active Scanning** | `AML.T0006` |
| Individuazione di Model Registry e artifact di training via API non protette | **Discover ML Artifacts** | `AML.T0007` |
| Rilevamento di token HF esposti e dipendenze esterne non verificate | **ML Supply Chain Compromise** | `AML.T0010` |
| Enumerazione della configurazione LLM, parametri e schemi API | **Discover ML Model Family** | `AML.T0014` |
| **Insieme completo delle attività di ricognizione sull'architettura AI** | **Reconnaissance Tactic** | `AML.TA0002` |

---

## 5. Case Study: Analisi della Campagna ShadowRay (CVE-2023-48022)

La campagna **ShadowRay** illustra l'impatto di una misconfigurazione di rete su un framework di calcolo distribuito per il Machine Learning.

### Origine e Vulnerabilità

* **Ray Framework:** La *Job Submission API* sulla porta `8265` del framework Ray è stata progettata nativamente priva di meccanismi di autenticazione integrati, presupponendo il deployment all'interno di reti private protette.
* **Esposizione pubblica:** L'assenza di controlli d'accesso a livello di rete ha consentito la rintracciabilità tramite i motori di ricerca per dispositivi connessi di oltre 230.000 dashboard Ray esposte su Internet.

### Meccanismo di Sfruttamento e Movimento Laterale

1. **Invocazione API:** L'invio di job non autenticati tramite l'endpoint `/api/jobs/` ha consentito l'esecuzione di script arbitrari sui nodi di calcolo.
2. **Reconnaissance Locale:** L'ispezione delle variabili d'ambiente (`printenv`) e del filesystem locale ha portato all'estrazione di credenziali cloud (AWS IAM Tokens) e segreti di sistema.
3. **Pivoting e Resource Hijacking:** Utilizzando i token cloud esfiltrati, gli attaccanti hanno effettuato un movimento laterale verso l'infrastruttura cloud collegata, allocando risorse GPU per il cryptomining (disattivando le segnalazioni di alto consumo e mascherando i processi di sistema).

### Evoluzione della Campagna (ShadowRay 2.0)

Nelle evoluzioni successive registrate tra la fine del 2025 e l'inizio del 2026, la campagna ha mostrato l'integrazione di tecniche di persistenza avanzate (servizi `systemd` e job `cron` nascosti) e l'impiego di strumenti per il degrado delle prestazioni di rete (TCP exhaustion), a dimostrazione di come la mancata protezione del piano di gestione d'inferenza e calcolo possa compromettere l'intero perimetro aziendale.