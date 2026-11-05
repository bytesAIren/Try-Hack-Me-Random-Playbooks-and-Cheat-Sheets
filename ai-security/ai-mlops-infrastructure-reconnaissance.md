# 🔍 AI & MLOps Infrastructure Reconnaissance: Port, Protocol & Endpoint Playbook

- **Dominio:** AI Infrastructure Reconnaissance, MLOps Attack Surface Mapping
- **Standard di Riferimento:** MITRE ATLAS (AML.T0002: Gather ML Artifacts, AML.T0000: ML Attack Staging)
- **Scopo:** Sintesi e mappatura dell'superficie di attacco per l'identificazione di servizi AI/ML esposti, database vettoriali e piattaforme di MLOps.

---

## 1. Mappa delle Porte & Protocolli Comuni nell'AI Stack

I sistemi di intelligenza artificiale espongono spesso porte non convenzionali per dividere il traffico web client (**HTTP/REST**) dal flusso binario ad alta velocità tra nodi di calcolo (**gRPC**).

### A. Model Serving (Inference Engines)

| Componente / Servizio | Porte Predefinite | Protocollo | Endpoint Tipici di Ricognizione | Impatto di Sicurezza se Esposto |
| :--- | :--- | :--- | :--- | :--- |
| **NVIDIA Triton Inference Server** | `8000`, `8001`, `8002` | HTTP, gRPC, Prometheus | `/v2/health/ready`<br>`/v2/models`<br>`/metrics` | Enumera i modelli in memoria, versioni, batch size e consumo GPU. |
| **vLLM Engine** | `8000` | HTTP | `/v1/models`<br>`/docs` | Espone API compatibile OpenAI. Permette inferenza non autorizzata o DoS. |
| **TorchServe (PyTorch)** | `8080`, `8081`, `8082` | HTTP | `/ping`<br>`/models`<br>`:8081/models` | La porta `8081` (Management API) consente spesso il caricamento remoto di nuovi modelli (`.mar`). |
| **TensorFlow Serving** | `8500`, `8501` | gRPC, HTTP | `/v1/models/<name>` | Consente il recupero dei metadati di versione del modello. |
| **Ollama** | `11434` | HTTP | `/api/tags`<br>`/api/show`<br>`/api/generate` | Espone la lista dei modelli scaricati in locale e permette l'esecuzione arbitraria di prompt. |

---

### B. Vector Databases (Memoria Semantica & RAG Pipelines)

| Componente | Porte Predefinite | Protocollo | Endpoint di Verifica | Impatto di Sicurezza |
| :--- | :--- | :--- | :--- | :--- |
| **Qdrant** | `6333`, `6334` | HTTP, gRPC | `/collections`<br>`/telemetry` | Espone le collezioni documentali e la dimensionalità dei vettori (es. file HR, contratti). |
| **Weaviate** | `8080` | HTTP, GraphQL | `/v1/schema`<br>`/v1/meta` | Consente l'interrogazione GraphQL non autenticata per scaricare documenti testuali interni. |
| **Milvus** | `19530` | gRPC | Connessione diretta client Milvus | Storage vettoriale distribuito: accesso diretto a record aziendali serializzati. |
| **ChromaDB** | `8000` | HTTP | `/api/v1/collections` | Database documentale RAG: dump completo delle collezioni di knowledge base aziendali. |

---

### C. Orchestration, Tracking & Notebooks

| Componente | Porte Predefinite | Protocollo | Endpoint di Verifica | Note & Rischi Noti |
| :--- | :--- | :--- | :--- | :--- |
| **MLflow Tracking** | `5000` | HTTP | `/api/2.0/mlflow/experiments/search`<br>`/#/experiments` | Espone tutti i parametri di training, dataset usati e link agli artifact archiviati nei bucket. |
| **Ray Cluster** | `8265`, `8000` | HTTP | `/api/jobs/`<br>`Ray Dashboard` | Bersaglio storico della campagna *ShadowRay*: esecuzione di comandi remoti e cryptomining non autorizzato. |
| **Kubeflow** | `80`, `443` | HTTP/S | `/pipeline/apis/v1beta1/pipelines` | Orchestrazione su Kubernetes: gestione pipeline di training e runner di calcolo. |
| **Jupyter Notebook** | `8888` | HTTP | `/api/kernels`<br>`/api/contents` | Se configurato con `--ip=0.0.0.0` e privo di token, garantisce accesso shell/terminale diretto al sistema. |
| **MinIO (Object Store)** | `9000`, `9001` | HTTP (S3-compatible) | `/minio/health/ready`<br>Console Web | Sposto ospita i file pesanti dei modelli (`.pkl`, `.onnx`, `.pt`). Spesso affetto da bucket listing non protetto. |

---

## 2. Metodologia di Scansione Focalizzata su AI Stack

Per identificare la presenza di servizi MLOps noti nell'infrastruttura, si monitorano le porte caratteristiche del settore:

```bash
# Scansione mirata sulle porte tipiche dei servizi AI / Vector DB / MLOps
nmap -p 5000,6333,6334,8000,8001,8002,8080,8081,8082,8265,8500,8501,8888,9000,9001,11434,19530 -sV -sC -oN ai_recon.txt <TARGET_IP>
```

---

## 3. Workflow di Interrogazione Manuale Rapida (Probe con curl)

Una volta identificate le porte aperte, si interrogano gli endpoint passivi per verificare lo stato dell'infrastruttura e identificare i metadati:

### NVIDIA Triton (Porta 8000 / 8002)

```bash
# 1. Verifica disponibilità del servizio
curl -s http://<TARGET_IP>:8000/v2/health/ready

# 2. Elenco dei modelli caricati in memoria
curl -s http://<TARGET_IP>:8000/v2/models

# 3. Metriche e nomi dei modelli esposti da Prometheus
curl -s http://<TARGET_IP>:8002/metrics | grep -i "model"
```

### Ollama (Porta 11434)

```bash
# 1. Elenco dei modelli installati
curl -s http://<TARGET_IP>:11434/api/tags

# 2. Dettagli sul modello specifico (architettura, template di prompt)
curl -s -X POST http://<TARGET_IP>:11434/api/show -d '{"name": "<MODEL_NAME>"}'
```

### Vector DB Qdrant (Porta 6333)

```bash
# Elenco delle collezioni semantiche attive (RAG)
curl -s http://<TARGET_IP>:6333/collections
```

### MLflow Tracking Server (Porta 5000)

```bash
# Interrogazione degli esperimenti e parametri di addestramento
curl -s http://<TARGET_IP>:5000/api/2.0/mlflow/experiments/search
```

### Jupyter Notebook (Porta 8888)

```bash
# Verifica se il notebook espone l'albero delle directory senza autenticazione
curl -s http://<TARGET_IP>:8888/api/contents
```

---

## 4. Checklist di Triage per l'Analista

Durante la fase di audit e verifica delle configurazioni dell'infrastruttura AI, verificare la presenza dei seguenti elementi critici:

- **Mancata Autenticazione sulle Management API:** Verificare se porte di gestione come TorchServe (`8081`), Ray (`8265`) o MLflow (`5000`) siano esposte e accessibili senza autenticazione.
- **Data Leakage nei Vector DB:** Controllare se le collezioni RAG siano liberamente interrogabili e contengano dati sensibili o riservati (es. `hr_confidential`, `internal_docs`, `financial_records`).
- **Rischio Deserializzazione (`.pkl`):** Se il registry di modelli consente il download di file `.pkl` (Python Pickle), la pipeline di caricamento è esposta a grave rischio di esecuzione remota di codice se importata senza firma crittografica.
- **Exposure di Metriche di Monitoraggio:** Gli endpoint `/metrics` di Prometheus (es. porte `8002`, `8082`) rivelano il fingerprinting completo dei modelli e dell'hardware sottostante (GPU/VRAM).