# 🗺️ AI Reconnaissance Methodology: 5-Phase Playbook, Detection & Surface Reduction

* **Dominio:** AI Infrastructure Reconnaissance, Threat Hunting, Attack Surface Management

* **Standard di Riferimento:** MITRE ATLAS (AML.TA0002: Reconnaissance, AML.T0006: Active Scanning, AML.T0007: Discover ML Artifacts, AML.T0010: ML Supply Chain Compromise, AML.T0014: Discover ML Model Family)

* **Scopo:** Guida metodologica sequenziale per l'esecuzione di audit di ricognizione su infrastrutture AI/ML, analisi delle firme nei log difensivi e contromisure immediate per la riduzione della superficie d'attacco.

## 1. Metodologia di Ricognizione AI in 5 Fasi

La procedura segue un approccio strutturato end-to-end, dalla raccolta di informazioni open-source all'ispezione della catena di fornitura.

```
[Fase 1: Ricognizione Passiva] ──> [Fase 2: Scansione Attiva] ──> [Fase 3: Fingerprinting API] ──> [Fase 4: Estrazione Metadati] ──> [Fase 5: Revisione Supply Chain]

```

---

### Fase 1: Ricognizione Passiva (Passive Reconnaissance)

Prima di interagire con le reti bersaglio, si analizzano le informazioni accessibili pubblicamente.

* **Search Engine per Dispositivi Connessi:** Ricerca su Shodan, Censys e FOFA dei banner dei servizi AI negli intervalli IP dell'organizzazione.

  * `port:5000 "MLflow"`

  * `port:8888 title:"Home Page - Select or create a notebook"`

  * `http.title:"Ray Dashboard"`

* **Dorking su Repository di Codice (GitHub):** Rilevamento di segreti o configurazioni esposte.

  * `filename:.env MLFLOW_TRACKING_URI`

  * `filename:.env HF_TOKEN`

  * `filename:config.json model_name site:github.com`

* **Letteratura e Blog Tecnici:** Ispezione di paper su arXiv o articoli tecnici pubblicati dall'azienda per identificare le architetture dei modelli e l'infrastruttura sottostante (*Mappatura MITRE ATLAS: AML.T0000 - Search for Victim's Publicly Available Research Materials*).

* **Container Registry & Offerte di Lavoro:** Analisi dei registry pubblici (DockerHub, GHCR) alla ricerca di immagini ML aziendali e ispezione degli annunci di lavoro (es. posizioni per "MLflow Administrator" o "Kubeflow Platform Engineer").

---

### Fase 2: Scansione Attiva (Active Scanning)

Mappatura delle porte e dei servizi MLOps tramite strumenti di port scanning focalizzati:

```bash
# Scansione mirata sulle porte tipiche dell'AI Stack
nmap -p 5000,6333,8000,8001,8002,8080,8265,8500,8501,8888,9000,11434,19530 -sV --script=http-title,http-headers <TARGET_IP>
```

* Particolare attenzione alle porte gRPC (`8001`, `8500`) che potrebbero essere identificate come servizi generici da Nmap (da approfondire con `grpcurl`).

* Verifica della presenza di endpoint di metriche Prometheus (`/metrics`) su porte dedicate (es. `8002` per Triton, `8082` per TorchServe).

---

### Fase 3: Fingerprinting delle API (API Fingerprinting)

Scansione delle rotte applicative tramite dizionari specifici per infrastrutture AI ed MLOps (es. con `ffuf` o `feroxbuster`).

#### Wordlist di Riferimento per Rotte AI:

```text
/v1/models
/v2/models
/v2/health/ready
/api/2.0/mlflow/experiments/list
/api/2.0/mlflow/registered-models/list
/pipeline/apis/v1beta1/pipelines
/api/serve/deployments/
/v1/schema
/v1/meta
/api/kernels
/api/contents
/openapi.json
/docs
/graphql
/metrics
/api/tags
/api/show
/collections
/healthz
/ping
```

Per ciascun endpoint con risposta `200 OK`, si analizzano gli header HTTP, la struttura JSON del corpo della risposta e i messaggi d'errore generati da richieste malformate.

---

### Fase 4: Estrazione dei Metadati (Metadata Extraction)

Interrogazione metodica dei servizi confermati per estrarre la mappa architetturale:

* **MLflow:** Estrazione di esperimenti, modelli registrati, versioni, URI degli artifact e ID utenti.

* **Inference Server (Triton / TF Serving):** Consultazione degli endpoint di configurazione dei modelli per estrarre specifiche dei tensori e backend di calcolo.

* **Vector Databases:** Ispezione degli endpoint di schema per identificare tipi di dati e modelli di embedding impiegati.

* **Jupyter Notebooks:** Enumerazione dei kernel attivi e del contenuto delle celle per l'individuazione di credenziali in chiaro.

---

### Fase 5: Revisione della Supply Chain (Supply Chain Review)

* Mappatura delle sorgenti di download dei modelli individuate nelle configurazioni e nei log di build dei container.

* Verifica dei permessi sui bucket di storage interni (`S3`, `GCS`, `MinIO`) contenenti gli artifact.

* Audit dei file di dipendenza (`requirements.txt`, `Pipfile`) per rilevare l'uso di nomi di pacchetti interni esposti al rischio di *dependency confusion* su PyPI.

---

## 2. Tabella dei Tool di Riferimento per Fase

| Strumento | Funzione Principale | Fase di Riferimento | 
| ----- | ----- | ----- | 
| **Shodan / Censys / FOFA** | Ricerca su scala globale dei banner dei servizi AI esposti su Internet. | **Fase 1** (Ricognizione Passiva) | 
| **GitHub Search / Dorks** | Individuazione di credenziali, token ed elementi di configurazione in repository pubblici. | **Fase 1** (Ricognizione Passiva) | 
| **Nmap (con script NSE)** | Discovery delle porte e identificazione delle versioni dei servizi. | **Fase 2** (Scansione Attiva) | 
| **grpcurl** | Interazione con servizi gRPC ed estrazione degli schemi Protobuf se la Reflection è attiva. | **Fase 2** (Scansione Attiva) | 
| **ffuf / feroxbuster** | Fuzzing delle directory tramite wordlist dedicate ai percorsi dell'AI stack. | **Fase 2, 3** (Scansione & Fingerprinting) | 
| **curl** | Interrogazione HTTP manuale, analisi degli header ed esecuzione di probe parlanti. | **Fase 3, 4** (Fingerprinting & Metadati) | 
| **MLOKit (IBM X-Force)** | Automazione dell'enumerazione di server MLflow ed esfiltrazione di metadati/modelli. | **Fase 4** (Estrazione Metadati) | 
| **Nuclei (ProjectDiscovery)** | Scansione basata su template per la rilevazione di misconfigurazioni note nei servizi AI. | **Fase 2, 3** (Scansione & Fingerprinting) | 
| **Agrus Scanner** | Scanner specifico per Shadow AI con probe su oltre 65.535 porte. | **Fase 2** (Scansione Attiva) | 

---

## 3. Prospettiva Difensiva: Rilevamento nei Log e nel SIEM

Tutte le attività di ricognizione generano pattern specifici nei log di sistema e di rete.

### Pattern Riconoscibili nei SIEM

* **Pattern di Enumerazione Modelli:** Raffiche di richieste `GET` sequenziali verso `/v2/models` da un singolo IP (es. da 10 a 50 richieste in pochi secondi verso lo stesso endpoint).

* **Accesso Scriptato a MLflow:** Chiamate API dirette a `/registered-models/list` e `/model-versions/search` prive di sessione o cookie generati dall'interfaccia web di MLflow (tipico di strumenti automatizzati come MLOKit).

* **Scraping delle Metriche Prometheus:** Richieste dirette all'endpoint `/metrics` provenienti da indirizzi IP esterni alla rete/CIDR del sistema di monitoraggio aziendale.

* **Port Scanning Focalizzato su AI:** Scansioni sequenziali da un singolo indirizzo IP sulle porte caratteristiche `5000`, `8000`, `8001`, `8080`, `8265`, `8888`.

* **Tentativi di Path Traversal:** Richieste contenenti sequenze `../` o `%2e%2e%2f` verso gli endpoint degli artifact di MLflow (indicatore di probing per CVE-2026-2033).

* **Accesso a Jupyter senza Sessione:** Richieste inviate agli endpoint `/api/kernels` e `/api/contents` in assenza di un cookie di sessione valido.

---

## 4. Quick Wins: Mitigazione e Riduzione della Superficie d'Attacco

Azioni immediate per ridurre la visibilità dell'infrastruttura AI agli occhi di uno scanner esterno:

1. **Abilitare l'Autenticazione su MLflow:** Configurare le variabili `MLFLOW_TRACKING_USERNAME` e `MLFLOW_TRACKING_PASSWORD` o posizionare il servizio dietro un reverse proxy con autenticazione.

2. **Proteggere le Istanze Jupyter:** Disabilitare i flag `--allow-root` e `--ip=0.0.0.0` negli ambienti di produzione. Imporre l'autenticazione tramite token e non vincolare l'ascolto su `0.0.0.0` senza un livello di ingress o VPN.

3. **Segregazione delle Porte a Livello Perimetrale:** Bloccare l'accesso diretto da Internet alle porte `5000`, `8000-8002`, `8080`, `8265`, `8500-8501`, `8888` e `9000`.

4. **Limitare i Controlli dei Modelli su Triton:** Avviare Triton con l'opzione `--model-control-mode none` in produzione per impedire il caricamento/scaricamento dinamico non autorizzato dei modelli.

5. **Restringere l'Accesso alle Metriche Prometheus:** Limitare l'accesso agli endpoint `/metrics` (porte `8002` e `8082`) al solo segmento di rete del sistema di monitoraggio aziendale.

6. **Rotazione e Granularità dei Token Hugging Face:** Impiegare token a grana fine (*fine-grained tokens*) con permessi di sola lettura e ambito ristretto.

7. **Rimozione degli Header Verbosi:** Rimuovere gli header di debug e i messaggi d'errore dettagliati dalle risposte delle API d'inferenza prima dell'instradamento verso reti esterne.

8. **Audit delle Policy sui Bucket di Storage:** Verificare che i bucket MinIO, S3 o GCS contenenti i pesi dei modelli non siano configurati con accesso in lettura pubblico.

---

## 5. Case Study: Compromissione di Hugging Face Spaces (Giugno 2024)

Nel giugno 2024, attori malevoli hanno ottenuto un accesso non autorizzato alla piattaforma **Hugging Face Spaces**, accedendo a segreti di autenticazione memorizzati dagli sviluppatori all'interno delle applicazioni ospitate.

### Lezione Appresa e Riconduzione alla Metodologia

* Tra i segreti sottratti figuravano token HF con permessi di lettura e scrittura su modelli e dataset privati.

* L'evento ha evidenziato l'impatto dei rischi analizzati nella **Fase 1** (Ricognizione Passiva) e nella **Fase 5** (Revisione della Supply Chain): le credenziali esposte in file `.env`, log di CI/CD o variabili d'ambiente diventano il vettore primario per la compromissione delle risorse cloud e dei modelli riservati.

* **Risposta e Mitigazione:** Hugging Face ha revocato i token impattati, raccomandando l'adozione di token a grana fine a portata limitata e sconsigliando la memorizzazione di chiavi statiche all'interno delle configurazioni applicative esposte.