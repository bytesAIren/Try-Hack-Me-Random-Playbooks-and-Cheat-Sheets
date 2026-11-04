# 🎯 Advanced AI & MLOps Fingerprinting: Techniques, Protocols & Threat Intelligence

- **Dominio:** AI Infrastructure Fingerprinting, Advanced Threat Intelligence & Reconnaissance
- **Standard di Riferimento:** MITRE ATLAS (AML.T0002: Gather ML Artifacts)
- **Scopo:** Guida metodologica e tecnica per l'identificazione di framework di intelligenza artificiale, runtime d'inferenza ed endpoint MLOps tramite analisi passive e attive.

---

## 1. HTTP Header Fingerprinting

Gli header di risposta HTTP costituiscono spesso il metodo più rapido e immediato per identificare il framework AI sottostante. Molti engine d'inferenza rivelano apertamente la propria identità a meno che non vengano esplicitamente mascherati dietro un reverse proxy.

### Indicatori di Header Principali

* **TorchServe:** Restituisce l'header univoco `Server: TorchServe/0.x.x`. Riconoscimento diretto senza ambiguità.
* **NVIDIA Triton Inference Server:** 
  * Inserisce l'header `NV-Status` nelle risposte HTTP.
  * Risponde a un header di richiesta specifico: inviando `endpoint-load-metrics-format: text`, Triton restituisce direttamente negli header di risposta la telemetria hardware (livelli di utilizzo di CPU e GPU).
  * *Nota:* Nessun altro framework condivide questo comportamento; la presenza di metriche GPU negli header HTTP indica univocamente Triton.
* **Servizi ML basati su FastAPI:** Mostrano l'header `Server: uvicorn`. La presenza di questo server abbinata a rotte come `/predict` o `/embeddings` indica chiaramente un backend ML sviluppato in Python.
* **Wrapper Compatibili con OpenAI (vLLM, LiteLLM, Ollama):** Restituiscono l'header `x-request-id` insieme a risposte JSON strutturate contenenti il campo `"object": "model"` interrogando l'endpoint `/v1/models`.

---

## 2. API Response Signatures (Firme del Payload JSON)

Ogni framework di inferenza formatta il corpo della risposta JSON in modo distintivo. L'analisi della struttura consente di identificare l'engine anche in assenza di header espliciti.

| Framework / Engine | Firma del Payload JSON (Response Body) | Caratteristica Identificativa |
| :--- | :--- | :--- |
| **TensorFlow Serving** | `{"model_version_status": [{"version": "1", "state": "AVAILABLE"}]}` | Struttura array rigida per lo stato della versione del modello. |
| **NVIDIA Triton** | `{"name": "fraud_detector", "versions": ["1"], "platform": "tensorflow_graphdef"}` | Esplicita sia il nome del modello sia la *platform* sottostante. |
| **OpenAI-Compatible**<br>*(vLLM, LiteLLM, Ollama)* | `{"object": "model", "id": "llama-3.1-8b", "created": 1700000000}` | Il campo `"object": "model"` identifica servizi conformi allo standard API OpenAI. |
| **MLflow Server** | Stack trace contenente namespace `mlflow.server` o `mlflow.tracking` | Anche nelle risposte d'errore rivela la presenza dell'infrastruttura. |

---

## 3. Error Message Fingerprinting (Fingerprinting tramite Gestione Errori)

La profilazione tramite messaggi d'errore è una delle tecniche più affidabili. Le API di inferenza AI sono rigide e si aspettano strutture tensoriali e tipi di dati estremamente precisi. L'invio intenzionale di un payload errato forza una risposta di errore parlante.

### Tecniche di Iniezione di Payload Malformati

* **TensorFlow Serving:** Inviando un array semplice di interi a un endpoint che richiede un oggetto tensoriale complesso, il server restituisce un errore contenente la stringa `tensorinfo_map` (stringa esclusiva di TF Serving).
* **MLflow:** Una richiesta malformata genera uno stack trace con riferimenti diretti ai namespace `mlflow.server`, `mlflow.tracking` o `databricks`.
  * *Vulnerabilità note:* Errori di Path Traversal in MLflow (es. **CVE-2024-1558**) possono rivelare i percorsi completi del filesystem del server.
* **Databricks Mosaic AI:** Token di autenticazione malformati restituiscono eccezioni Java esplicite come `io.jsonwebtoken.IncorrectClaimException`.

> **Perché funziona:** Durante lo sviluppo delle pipeline di Machine Learning, i Data Scientist necessitano di dettagli di debug estesi. Tali configurazioni verbose rimangono spesso attive di default anche nei deployment di produzione.

---

## 4. Convenzioni di Naming degli Endpoint

A differenza delle API REST tradizionali che utilizzano sostantivi relativi alle risorse (`/users`, `/accounts`), gli endpoint AI utilizzano termini legati ad azioni computazionali.

### Glossario delle Rotte per Directory Brute-Forcing

* **Endpoint di Inferenza:** `/predict`, `/invocations` (standard AWS SageMaker), `/infer`, `/generate`, `/embeddings`, `/score`
* **Gestione dei Modelli:** `/v1/models`, `/v2/models`
* **API Interne MLflow:** `/api/2.0/mlflow/` (prefisso distintivo ed esclusivo)
* **Pipeline Kubeflow:** `/pipeline/apis/v1beta1/`

*Consiglio Operativo:* Includere queste rotte nelle wordlist personalizzate di strumenti come `ffuf` o `feroxbuster`, in quanto i dizionari standard (es. SecLists) spesso non le includono.

---

## 5. gRPC Fingerprinting

Molti servizi d'inferenza espongono l'interfaccia gRPC affiancata a quella HTTP per ridurre le latenze.

* **NVIDIA Triton:** Porta gRPC predefinita `8001`
* **TensorFlow Serving:** Porta gRPC predefinita `8500`

I tradizionali scanner HTTP non rilevano questi servizi in quanto gRPC è un protocollo binario. Attraverso lo strumento `grpcurl`, se la *gRPC Reflection* è abilitata, è possibile ricavare l'intero schema Protobuf:

```bash
# 1. Elenco dei servizi RPC disponibili
grpcurl -plaintext <TARGET_IP>:8001 list

# 2. Descrizione dettagliata e schema dell'interfaccia di inferenza
grpcurl -plaintext <TARGET_IP>:8001 describe inference.GRPCInferenceService
```

*Risultato:* L'output rivela tutti gli RPC esposti, la struttura esatta di ogni tensore di input e il formato dell'output (equivalente al recupero di un file `/openapi.json` su API REST).

---

## 6. TLS Fingerprinting (Impronte JA3 / JA4)

Il traffico interno tra microservizi e pipeline ML è dominato da librerie Python (`requests`, `urllib`, client gRPC) anziché da browser web tradizionali, traducendosi in impronte TLS distinte.

* L'analisi degli hash **JA3** e **JA4** permette di distinguere a livello di rete il traffico automatizzato delle pipeline ML dalla navigazione umana.
* L'analisi delle campagne reali mostra che oltre il **99% del traffico di scansione** in determinate campagne condivide la medesima firma **JA4H**, pur provenendo da decine di indirizzi IP differenti distribuiti globalmente, rivelando l'uso della stessa attrezzatura automatizzata.

---

## 7. Case Study: GreyNoise LLM Endpoint Reconnaissance

Tra ottobre 2025 e gennaio 2026, la rete di honeypot Ollama di **GreyNoise** ha intercettato oltre 91.000 sessioni di attacco dirette a infrastrutture AI.

### Dinamica dell'Attacco

In una specifica campagna durata 11 giorni (iniziata il 28 dicembre), un attaccante ha inviato più di **80.000 richieste** probing verso endpoint conversazionali per verificare la compatibilità con le API di OpenAI, Gemini o Anthropic. L'attività ha preso di mira oltre 73 diversi modelli (varianti GPT-4o, Claude, Llama, Gemini, DeepSeek, Mistral, Qwen e Grok).

### Metodologia di Reconnaissance

L'attaccante ha impiegato prompt innocui privi di payload malevoli:
* `"hi"` (32.716 occorrenze)
* `"How many states are there in the United States?"` (27.778 occorrenze)
* `"How many letter 'r' are in the word strawberry?"`

**Meccanismo:** Ogni modello o gateway risponde con strutture, formati e contenuti leggermente diversi a seconda dell'architettura e del proxy. Confrontando la risposta con gli schemi API noti, l'attaccante ha identificato con precisione il modello e la configurazione sottostante senza attivare i filtri di Content Moderation.

### Valutazione di Threat Intelligence

L'infrastruttura di attacco ha mostrato sovrapposizioni con IP precedentemente impiegati per lo sfruttamento di oltre 200 CVE note (inclusa la vulnerabilità **React2Shell - CVE-2025-55182**).

> **Conclusione:** Un attore malevolo strutturato ha condotto una ricognizione sistematica su scala globale, senza eseguire exploitation diretta o inviare payload malevoli, al solo scopo di mappare l'infrastruttura esposta e creare elenchi di bersagli da colpire nelle fasi successive della kill chain.