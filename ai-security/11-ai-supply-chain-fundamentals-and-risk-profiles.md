# 📦 AI Supply Chain Security: Fondamenti, Relazioni di Fiducia e Profili di Rischio

* **Dominio:** AI Supply Chain Security, Model Integrity, MLOps Security, File Format Analysis
* **Standard di Riferimento:** MITRE ATLAS (AML.T0010: ML Supply Chain Compromise), OWASP Top 10 for LLMs (LLM03: Supply Chain Vulnerabilities)
* **Scopo:** Guida metodologica e concettuale per comprendere le differenze tra supply chain del software tradizionale e dell'AI, i due paradigmi di consumo dei modelli (Download vs API) e i rispettivi profili di rischio.

---

## 1. Dal Codice ai Modelli: Un Nuovo Artefatto nella Supply Chain

Le catene di fornitura del software tradizionale gestiscono principalmente **codice sorgente** (librerie, framework e pacchetti scritti da sviluppatori). Le catene di fornitura dell'Intelligenza Artificiale introducono un artefatto fondamentalmente differente: **i modelli addestrati**.

Un modello addestrato non è semplicemente codice. È il risultato dell'interazione di quattro elementi:

1. **Architettura:** La struttura della rete neurale.
2. **Dati di Addestramento:** Milioni di esempi utilizzati durante il training.
3. **Processo di Addestramento:** Configurazione dell'hardware, iperparametri e impostazioni.
4. **Pesi Serializzati (*Serialised Weights*):** I parametri appresi memorizzati all'interno di un file.

```
┌────────────────────────────────────────────────────────┐
│                   UN MODELLO ADDESTRATO                │
├────────────────────────────────────────────────────────┤
│ 1. Architettura della Rete Neurale                     │
│ 2. Dataset di Pre-Training (Milioni di Esempi)         │
│ 3. Processo e Hardware di Addestramento                 │
│ 4. Pesi Serializzati (Parametri appresi salvati su File)│
└────────────────────────────────────────────────────────┘
```

### L'Analogia del Salvataggio di Gioco

I pesi serializzati possono essere paragonati al file di salvataggio di un videogioco: catturano esattamente lo stato in cui vi trovate (posizione, inventario, missioni completate). I pesi di un modello catturano tutto ciò che l'AI ha "imparato".

> Scaricare un modello pre-addestrato da una fonte sconosciuta equivale a caricare il salvataggio di uno sconosciuto: il gioco parte regolarmente e il personaggio ha le statistiche corrette, ma l'autore potrebbe aver incorporato codice invisibile all'interno del file.

A differenza del codice tradizionale che viene eseguito così com'è scritto, i file di modello basati su **Pickle** (`.pkl`, `.bin`) contengono oggetti serializzati in grado di **eseguire codice arbitrario non appena viene invocata la funzione `model.load()`**.

---

## 2. Il Rischio del Transfer Learning e del Fine-Tuning

La maggior parte dei team non addestra modelli da zero (procedura che richiede settimane e notevoli risorse computazionali), ma adotta la tecnica del **Transfer Learning**, in particolare il **Fine-Tuning**: si prendono i pesi di un modello base e li si adatta su un dataset più piccolo e specifico per il proprio task.

Questo approccio crea una forte tensione a livello di sicurezza:

```
[ Modello Base Sconosciuto ] ───(Pre-Training su Dati Non Verificati)───┐
                                                                          │
                                                                          ▼
[ Applicazione Finale ] ◄───(Fine-Tuning su Dati Aziendali) ──────────────┘
```

* **Persistenza delle Backdoor:** Ricerche scientifiche dimostrano che le *backdoor* inserite durante la fase di pre-training possono **sopravvivere al processo di fine-tuning**. Un attaccante che avvelena un modello base popolare non compromette una sola applicazione, ma tutte le applicazioni a valle che eseguono il fine-tuning da esso.

* **Adattatori LoRA:** Metodi moderni di fine-tuning come **LoRA** (*Low-Rank Adaptation*) consentono di condividere piccoli file adattatori da agganciare al modello base. La supply chain presenta così due dipendenze di fiducia: il modello base e l'adattatore.

> **Regola Fondamentale:** Il fine-tuning adatta il comportamento esterno del modello, ma **non può rimuovere ciò che è stato incorporato durante la fase di pre-training**. Un modello base pulito combinato con un adattatore malevolo (o viceversa) dà origine a un sistema compromesso.

---

## 3. I Quattro Componenti della AI Supply Chain

La catena di fornitura dell'IA si compone di quattro elementi chiave, ciascuno dei quali introduce specifiche relazioni di fiducia e superfici d'attacco:

| Componente | Definizione | Origine Tipica | Esempi |
| :--- | :--- | :--- | :--- |
| **Modelli (Models)** | Pesi pre-addestrati e architetture neurali. | Hugging Face Hub, TensorFlow Hub, PyTorch Hub | `bert-base-uncased`, `gpt2`, `llama-3-8b` |
| **Dataset (Datasets)** | Dati utilizzati per l'addestramento e la valutazione. | Hugging Face Datasets, Kaggle, Repository Accademici | `ImageNet`, `Common Crawl` |
| **Framework** | Librerie ML utilizzate per addestrare ed eseguire i modelli. | PyPI, Conda, GitHub | `PyTorch`, `TensorFlow`, `scikit-learn` |
| **Dipendenze (Dependencies)** | Pacchetti di supporto necessari al funzionamento dei framework. | PyPI, npm, Conda-Forge | `NumPy`, `SciPy`, `Pillow`, `tokenizers` |

---

## 4. Architettura delle Relazioni di Fiducia

In un tipico deployment di Machine Learning, i quattro componenti si integrano secondo una catena di dipendenze sequenziali:

```
[ Applicazione AI ]
        │
        ▼ (Si fida del)
[ Framework ML (PyTorch / TensorFlow) ]
        │
        ▼ (Si fida del)
[ Package Registry (PyPI / Conda) ]
        │
        ▼ (Si fida del)
[ Maintainer / Uploader Terzo ]
```

Ogni freccia all'interno dell'architettura rappresenta una **relazione di fiducia implicita** verso componenti sviluppati da terzi:

1. L'applicazione si fida del modello e del framework ML.
2. Il framework ML si fida del gestore dei pacchetti (PyPI/Conda).
3. Il gestore dei pacchetti si fida dell'utente o dell'entità che ha caricato la dipendenza.

Se anche una sola di queste relazioni di fiducia viene compromessa (es. tramite *dependency confusion*, *model poisoning* o *pickle execution*), l'intero sistema applicativo a monte risulta compromesso.

---

## 5. I Due Paradigmi di Consumo dei Modelli AI

Non tutte le catene di fornitura dell'Intelligenza Artificiale presentano la medesima struttura. Le organizzazioni consumano i modelli di Machine Learning principalmente secondo due paradigmi architetturali distinti, ciascuno caratterizzato da uno specifico profilo di rischio.

```
┌────────────────────────────────────────────────────────┐
│                   PARADIGMA 1: DOWNLOAD                │
│ (Il file del modello attraversa il perimetro e gira    │
│  sull'infrastruttura locale)                           │
└───────────────────────────┬────────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────────┐
│                   PARADIGMA 2: API                     │
│ (Solo la risposta JSON attraversa il perimetro; la     │
│  supply chain del provider è invisibile)              │
└────────────────────────────────────────────────────────┘
```

---

### 5.1 Paradigma 1: Download dei File di Modello (Self-Hosted Inference)

In questo approccio tradizionale, l'organizzazione scarica i pesi pre-addestrati (`.pkl`, `.safetensors`, `.pt`, `.gguf`, ecc.) da repository pubblici o privati (es. Hugging Face Hub) e li carica sulla propria infrastruttura.

L'organizzazione è direttamente responsabile dell'hosting, dell'inferenza e della sicurezza del sistema operativo.

#### Profilo di Rischio per Formato di File

Tutti i file caricati attraversano il confine di fiducia (*trust boundary*) ed eseguono codice o calcoli sui server interni. Il formato del file determina il tipo di vulnerabilità a cui l'infrastruttura è esposta:

| Formato File | Meccanismo di Archiviazione | Rischio di Esecuzione Codice (RCE) | Note di Sicurezza |
| :--- | :--- | :--- | :--- |
| **`.pkl` / `.pt` / `.bin`** | Serializzazione Python Pickle | **CRITICO (RCE Immediata)** | Consentono l'esecuzione di codice Python arbitrario non appena viene invocata la funzione `model.load()`. |
| **`.safetensors`** | Valori numerici grezzi dei pesi | **Basso (Nessuna RCE)** | Progettato per eliminare i rischi di serializzazione. Memorizza unicamente tensori senza codice eseguibile. |
| **`.h5`** | Formato nativo Keras (HDF5) | **MEDIO (RCE Architetturale)** | Non basato su Pickle, ma può contenere codice eseguibile personalizzato incorporato nelle definizioni dei layer. |
| **`.gguf`** | Formato binario ottimizzato per LLM locali | **BASSO (No Serializzazione)** | Formato standard per modelli locali (LLaMA, Mistral). Non soffre di exploit di serializzazione, ma rimane soggetto a *backdoor nei pesi*. |

#### Il Rischio della Quantizzazione (Quantisation)

I modelli `.gguf` sono quasi sempre soggetti a **quantizzazione**, un processo che comprime i pesi del modello convertendoli da numeri in virgola mobile a 32-bit (FP32) a interi a 4-bit o 8-bit (INT4/INT8), riducendo le dimensioni del file per consentire l'esecuzione su hardware commerciale.

> **Punto Critico di Supply Chain:** Se la compressione viene eseguita da terzi, la fase di quantizzazione rappresenta un passaggio non verificato della supply chain: ci si fida non solo dell'addestramento originale, ma di ogni trasformazione che il file ha subito prima del download.

---

### 5.2 Paradigma 2: Invocazione di Modelli via API (Hosted Services)

Sempre più organizzazioni utilizzano modelli AI forniti da provider gestiti (OpenAI, Anthropic, Google) o da aggregatori (OpenRouter). L'applicazione invia un prompt e riceve una risposta testo/JSON; l'organizzazione non entra mai in contatto diretto con il file del modello.

Questo approccio elimina i rischi legati alla deserializzazione (nessun file `.pkl` da caricare), ma introduce una **supply chain opaca**.

#### Matrice Comparativa delle Relazioni di Fiducia

| Elemento di Supply Chain | Paradigma Download (Self-Hosted) | Paradigma API (Hosted Service) |
| :--- | :--- | :--- |
| **Pesi del Modello** | Ispezionabili e scansionabili localmente. | Controllati interamente dal provider; non ispezionabili. |
| **Dataset di Addestramento** | Spesso documentato nelle *Model Card*. | Spesso non divulgato o descritto in modo vago. |
| **Fine-Tuning** | Gestito e controllato dall'organizzazione. | Il provider può aggiornare o rifinire il modello senza preavviso. |
| **System Prompts** | Definiti internamente dallo sviluppatore. | Template o infrastrutture terze possono alterare il comportamento. |
| **Versioning** | È possibile fissare l'hash univoco del file (`SHA-256`). | Il provider può aggiornare il modello mantenendo lo stesso endpoint API. |
| **Hosting & Ambiente** | Infrastruttura proprietaria isolata. | Infrastruttura condivisa del provider (Multi-Tenant). |

---

## 6. Sintesi e Considerazioni di Sicurezza

```
┌────────────────────────────────────────────────────────────────────────┐
│ KEY INSIGHT:                                                           │
│ Quando si invoca un modello via API, si sceglie di fidarsi dell'intera  │
│ pipeline del provider: gestione dei dati di training, scelte di      │
│ fine-tuning, sicurezza dell'infrastruttura ed evoluzione dei modelli. │
│ La supply chain non scompare; viene semplicemente nascosta dietro      │
│ una chiamata API.                                                      │
└────────────────────────────────────────────────────────────────────────┘
```

1. **Scelta del Paradigma Download:** Richiede strumenti di scansione dei file (`safetensors-convert`, `picklescan`), isolamento degli ambienti di inferenza e verifica dell'integrità tramite hash crittografici.

2. **Scelta del Paradigma API:** Richiede contratti di livello di servizio (SLA), audit delle policy di privacy dei dati del provider, gestione rigorosa delle chiavi API e monitoraggio continuo delle risposte per rilevare variazioni non dichiarate nei modelli sottostanti.
