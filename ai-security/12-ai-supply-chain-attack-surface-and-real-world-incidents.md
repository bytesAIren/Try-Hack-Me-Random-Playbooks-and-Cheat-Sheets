# 🎯 AI Supply Chain Attack Surface: 4 Livelli d'Attacco e Incidenti Reali (2022–2025)

* **Dominio:** AI Supply Chain Security, Threat Surface Mapping, Real-World Incident Analysis, Threat Intelligence
* **Standard di Riferimento:** MITRE ATLAS (AML.T0010: ML Supply Chain Compromise, AML.T0018: Backdoor ML Model), OWASP Top 10 for LLMs (LLM03: Supply Chain Vulnerabilities)
* **Scopo:** Analisi dettagliata della superficie d'attacco nel paradigma di download dei modelli AI, classificazione su 4 livelli, studio dell'effetto combinato (*Compounding Effect*) e analisi di cinque incidenti reali con lezioni apprese.

---

## 1. Mappatura della Superficie d'Attacco

Nel paradigma di download dei modelli, ogni tappa lungo la catena di fornitura rappresenta un potenziale punto di intercettazione. Un attaccante non necessita di controllare l'intera infrastruttura: gli è sufficiente compromettere un singolo punto di snodo (*checkpoint*).

```
[ Layer 4: Infrastruttura ]  ──► Compromissione Account / CI/CD Pipeline
         │
         ▼
[ Layer 3: Dati ]            ──► Poisoning del Dataset (anche solo 0.1%)
         │
         ▼
[ Layer 2: Dipendenze ]      ──► Typosquatting / Dependency Confusion
         │
         ▼
[ Layer 1: Modelli ]         ──► Code Execution (Pickle) / Backdoor nei Pesi
```

---

## 2. Analisi Dettagliata dei 4 Livelli d'Attacco

### Layer 1: Il Livello dei Modelli (Model Layer)

Rappresenta l'elemento più distintivo della superficie d'attacco dell'IA. Gli attaccanti possono inserire codice eseguibile nei file, modificare l'architettura della rete neurale o alterare sottilmente i pesi appresi per introdurre comportamenti malevoli nascosti.

#### Tassonomia degli Attacchi a Livello di Modello

Non tutti gli attacchi su questo livello operano allo stesso modo; si distinguono tre sotto-livelli con comportamenti differenti:

```
┌────────────────────────────────────────────────────────┐
│ 1. Livello di Serializzazione (Serialisation-Level)    │
│    • Codice nascosto nel formato del file (.pkl, .pt)  │
│    • Si esegue all'atto del caricamento (model.load())  │
├────────────────────────────────────────────────────────┤
│ 2. Livello di Architettura (Architecture-Level)        │
│    • Logica malevola inserita nei layer della rete     │
│    • Si esegue a ogni singola inferenza/predizione     │
├────────────────────────────────────────────────────────┤
│ 3. Livello dei Pesi (Weights-Level)                    │
│    • Valori appresi alterati per creare backdoor       │
│    • Attiva comportamenti malevoli su trigger specifici│
└────────────────────────────────────────────────────────┘
```

> **Key Insight:** Convertire un modello in un formato sicuro (es. da `.pkl` a `.safetensors`) elimina completamente gli attacchi al livello di serializzazione, ma **lascia del tutto inalterati gli attacchi al livello di architettura e di pesi**. Una difesa efficace richiede ispezioni su tutti e tre i livelli.

---

### Layer 2: Il Livello delle Dipendenze (Dependency Layer)

Riguarda i pacchetti e le librerie da cui il progetto ML dipende direttamente o indirettamente.

* **Vettori d'attacco:** *Dependency Confusion*, *Typosquatting* (pubblicazione di pacchetti con nomi simili a quelli ufficiali) e caricamento su registri pubblici (PyPI, npm, Conda-Forge) di librerie malevole correlate da documentazioni dall'aspetto professionale che occultano malware.

---

### Layer 3: Il Livello dei Dati (Data Layer)

I dati di addestramento costituiscono le fondamenta di qualsiasi modello di Machine Learning e il loro avvelenamento (*Data Poisoning*) è estremamente complesso da rilevare.

* **Soglia di Attivazione Backdoor:** Ricerche di settore dimostrano che la sostituzione di appena il **0,1%** di un dataset di addestramento con campioni artefatti è sufficiente per introdurre una *backdoor* affidabile, senza influire in modo misurabile sull'accuratezza generale dei dati puliti.

* **Soglia per Denial of Service (DoS):** Per attacchi mirati a degradare le prestazioni complessive, la percentuale utile si abbassa fino allo **0,001%**.

---

### Layer 4: Il Livello dell'Infrastruttura (Infrastructure Layer)

Riguarda i sistemi che ospitano, distribuiscono e gestiscono gli artefatti di Intelligenza Artificiale.

* **Vettori d'attacco:** Compromissione degli account sui repository di modelli (es. Hugging Face Hub), iniezione di passaggi malevoli nelle pipeline CI/CD di build, o furto delle credenziali dei maintainer per distribuire aggiornamenti malevoli sfruttando identità fidate.

---

## 3. L'Effetto Combinato (The Compounding Effect)

Questi quattro livelli non operano in modo isolato. Un attacco strutturato può combinare tecniche su più livelli contemporaneamente per massimizzare la penetrazione:

```
                  ┌─────────────────────────────────────────┐
                  │ Account Hugging Face Compromesso        │ (Infrastructure Layer)
                  └────────────────────┬────────────────────┘
                                       │
                                       ▼
                  ┌─────────────────────────────────────────┐
                  │ Modello Backdoor con Codice Pickle      │ (Model Layer)
                  └────────────────────┬────────────────────┘
                                       │
                                       ▼
                  ┌─────────────────────────────────────────┐
                  │ requirements.txt con Typosquatting      │ (Dependency Layer)
                  └────────────────────┬────────────────────┘
                                       │
                                       ▼
                  ┌─────────────────────────────────────────┐
                  │ Dataset Avvelenato Distribuito          │ (Data Layer)
                  └─────────────────────────────────────────┘
```

> **Conclusione:** Un singolo download da una fonte non verificata o compromessa può attivare simultaneamente vettori d'attacco su tutti e quattro i livelli dell'infrastruttura aziendale.

---

## 4. Incidenti Reali: L'Inganno al Cuore della Supply Chain

Le tecniche di attacco alla catena di fornitura dell'IA non sono teoriche: sono state utilizzate in attacchi reali contro sviluppatori, ricercatori e organizzazioni.

> **Principio Fondamentale:** Un modello che appare affidabile in superficie (con valutazioni elevate, molte stelle o documentazione curata) può occultare codice malevole al proprio interno. La reputazione o le metriche di gradimento di una piattaforma non costituiscono una garanzia di sicurezza.

```
┌────────────────────────────────────────────────────────┐
│                   ISPEZIONE IN SUPERFICIE              │
│ • Valutazioni elevate / Download elevati              │
│ • Documentazione professionale / Schede modello curati │
├────────────────────────────────────────────────────────┤
│                   REALTÀ INTERNA (PAYLOAD)            │
│ • Codice eseguibile occultato (Pickle Injection)       │
│ • Backdoor nei pesi / Evasione dei filtri di scansione │
└────────────────────────────────────────────────────────┘
```

---

### 4.1 PyTorch Dependency Confusion (Dicembre 2022)

Il giorno di Natale del 2022, un attore malevolo ha pubblicato sul registro ufficiale PyPI un pacchetto malevolo denominato `torchtriton`.

* **Meccanismo di Attacco:** Sfruttamento della *Dependency Confusion*. Le *nightly build* ufficiali di PyTorch dipendevano da un pacchetto interno con lo stesso nome (`torchtriton`). La logica di risoluzione delle versioni di `pip` ha preferito la versione pubblica malevola presente su PyPI rispetto a quella del registro privato.

* **Payload ed Esfiltrazione:** Il pacchetto malevolo esfiltrava chiavi SSH, file di configurazione Git, il file `/etc/passwd` e fino a 1.000 file dalle home directory degli utenti. I dati venivano inviati all'esterno tramite query DNS cifrate.

* **Tempo di Persistenza:** Il pacchetto è rimasto attivo e scaricabile per cinque giorni prima della scoperta e rimozione.

* *Riferimento:* Blog Ufficiale PyTorch - *"Compromised nightly dependency"* (Dicembre 2022).

---

### 4.2 Vulnerabilità sul Hub di Hugging Face (2023–2024)

Molteplici criticità sono emerse sulla principale piattaforma mondiale di hosting per modelli di Machine Learning.

* **Novembre 2023 (Lasso Security):** Ricercatori hanno scoperto oltre 1.500 token API di Hugging Face esposti pubblicamente. Ben 655 di questi token disponevano di permessi di scrittura, consentendo a terzi non autorizzati di modificare i repository ufficiali e inserire codice o modelli compromessi.

* **2024 (JFrog Security Research):** Identificazione di circa 100 modelli basati sul formato `.pkl` (Pickle) contenenti malware. I modelli si presentavano con descrizioni professionali e offrivano funzionalità ML reali, ma eseguivano silenziosamente codice malevole all'atto del caricamento (`model.load()`).

* *Riferimenti:* Lasso Security - *"1,500 HuggingFace API tokens were exposed"* (Nov 2023) / JFrog Research - *"Data scientists targeted by malicious Hugging Face ML models"* (2024).

---

### 4.3 Compromissione della Pipeline di Build di Ultralytics (Dicembre 2024)

Un attacco mirato alla catena di integrazione e distribuzione continua (CI/CD) della libreria di computer vision **YOLO**.

* **Meccanismo di Attacco:** Gli attaccanti hanno compromesso i workflow di **GitHub Actions** dell'organizzazione Ultralytics.

* **Impatto:** L'iniezione di codice malevole nel processo di build ha provocato l'incorporamento di un software di cryptomining direttamente all'interno dei pacchetti pubblicati su PyPI. Gli sviluppatori che eseguivano un normale comando `pip install ultralytics` scaricavano automaticamente la versione malevola.

* **Lezione:** L'attacco ha preso di mira l'infrastruttura automatizzata di build anziché il codice sorgente nei repository.

* *Riferimento:* PyPI Blog - *"Ultralytics attack analysis"* (Dicembre 2024).

---

### 4.4 Compromissione del Pacchetto `@solana/web3.js` (Dicembre 2024)

Sebbene non specifico per l'IA, questo incidente evidenzia una dinamica identica a quella che affligge le supply chain MLOps.

* **Meccanismo di Attacco:** Furto delle credenziali npm di un maintainer del pacchetto. Gli attaccanti hanno pubblicato due versioni contenenti una backdoor ideata per esfiltrare le chiavi private dei portafogli di criptovaluta.

* **Danno Economico:** Le perdite stimate hanno raggiunto tra i 130.000\$ e i 184.000\$ nelle poche ore precedenti la rilevazione.

* **Lezione:** Identità fidata → compromissione a monte → vittime che non hanno commesso alcun errore di configurazione diretto.

* *Riferimento:* Anza Security - *"web3.js exploit root cause analysis"* (Dicembre 2024).

---

### 4.5 Evasione dei Scanner "NullifAI" su Hugging Face (Febbraio 2025)

I ricercatori di ReversingLabs hanno scoperto una tecnica di bypass contro gli scanner di sicurezza automatizzati della piattaforma Hugging Face.

* **Meccanismo di Attacco:** Creazione di file `.pkl` intenzionalmente corrotti. Lo scanner automatizzato della piattaforma falliva nell'analisi a causa di eccezioni nel parsing delle strutture Pickle malformate, contrassegnando il file come sicuro.

* **Esecuzione:** Quando il modello veniva scaricato e caricato da uno sviluppatore in ambiente Python, l'interprete eseguiva comunque il payload malevole contenuto nel file.

* **Lezione:** Un modello che supera i controlli automatici di una piattaforma non è necessariamente privo di rischi.

* *Riferimento:* ReversingLabs - *"RL identifies malware ML model hosted on Hugging Face"* (Febbraio 2025).

---

## 5. Matrice Comparativa degli Incidenti

| Incidente | Anno | Livello della Supply Chain Colpito | Vettore d'Attacco Primario | Impatto Principale |
| :--- | :--- | :--- | :--- | :--- |
| **PyTorch `torchtriton`** | 2022 | **Dependency Layer** | Dependency Confusion (PyPI) | Esfiltrazione di chiavi SSH, file di configurazione e dati `/etc/passwd`. |
| **Hugging Face Token Leak** | 2023 | **Infrastructure Layer** | Esposizione di API Tokens con permessi di scrittura | Rischio di manomissione diretta di repository ufficiali e modelli fidati. |
| **Hugging Face Malicious Pickle** | 2024 | **Model Layer** | Pickle Code Injection (`.pkl`) | Esecuzione di codice arbitrario all'invocazione di `model.load()`. |
| **Ultralytics (YOLO)** | 2024 | **Infrastructure / Build Layer** | Compromissione workflow GitHub Actions | Distribuito cryptominer all'interno del pacchetto PyPI ufficiale. |
| **`@solana/web3.js`** | 2024 | **Infrastructure Layer** | Hijacking dell'account Maintainer (npm) | Esfiltrazione di chiavi private e perdite finanziarie immediate. |
| **NullifAI Evasion** | 2025 | **Model / Infrastructure Layer** | Malformed Pickle Evasion su scanner HF | Bypass completo dei controlli automatizzati di piattaforma. |

---

## 6. Conclusioni per la Sicurezza delle Infrastrutture AI

L'analisi degli incidenti reali e della superficie d'attacco su 4 livelli sottolinea tre regole fondamentali per la protezione della supply chain dell'IA:

1. **Non affidarsi esclusivamente agli scanner di piattaforma:** Gli attaccanti studiano e aggirano attivamente le regole degli strumenti di scansione automatizzata.

2. **Privilegiare formati di modelli sicuri:** Convertire e adottare formati privi di serializzazione esplicita (es. `.safetensors`) riduce la superficie d'attacco legata all'esecuzione remota di codice.

3. **Verificare e bloccare le dipendenze:** Utilizzare file di lock (`requirements.txt`, `Pipfile.lock`), hashes specifici e registri privati protetti per prevenire attacchi di tipo *dependency confusion*.
