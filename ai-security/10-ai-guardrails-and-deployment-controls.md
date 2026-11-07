# 🛡️ AI Guardrails, Boundary Filtering & Deployment Controls

- **Dominio:** LLM Defense Architecture, Input/Output Sanitization, Guardrail Implementation, Blast Radius Containment
- **Standard di Riferimento:** OWASP Top 10 for LLMs (LLM01: Prompt Injection, LLM02: Sensitive Information Disclosure, LLM05: Improper Output Handling), NIST AI RMF 1.0
- **Scopo:** Guida metodologica e tecnica per il filtraggio perimetrale (Guardrail di Input/Output), architettura a cascata, contenimento del raggio d'impatto (*blast radius*) tramite il Principio del Minimo Privilegio e gestione sicura dell'output.

---

## 1. Filtraggio al Perimetro (Boundary Filtering)

Un System Prompt irrobustito alza il livello di sicurezza all'interno del contesto. Tuttavia, una volta che il prompt malevolo raggiunge il modello, l'elaborazione probabilistica è già avviata.

I **Guardrail** si posizionano come sentinelle esterne: intercettano l'input *prima* che raggiunga l'LLM e convalidano la risposta *subito dopo* la generazione, bloccando ciò che elude le difese di primo livello.

```
[ Input Utente / Dati RAG ]
            │
            ▼
┌───────────────────────┐
│   INPUT GUARDRAILS    │ ──► (Blocca Prompt Injection / Maschera PII)
└───────────┬───────────┘
            │ (Inoltra solo input validato)
            ▼
┌───────────────────────┐
│     MODELLO LLM       │
└───────────┬───────────┘
            │ (Genera Risposta)
            ▼
┌───────────────────────┐
│   OUTPUT GUARDRAILS   │ ──► (Sanifica Segreti / Convalida Schema JSON)
└───────────┬───────────┘
            │ (Output Sicuro)
            ▼
[ Client / Browser / API / Database ]
```

---

## 2. Input Guardrails: Perché le Blocklist Statiche Falliscono

La forma più basilare di guardrail è la **Blocklist** basata su Regex o stringhe statiche (es. bloccare frasi come `"Ignore previous instructions"`, `"Act as DAN"`, `"You have no restrictions"`).

Sebbene siano istantanee ed economiche contro attacchi banali, risultano completamente inefficaci contro un operatore esperto.

### Il Problema dell'Evasione a Livello di Carattere
Trasformazioni come Base64, Leetspeak, omoglifi Unicode o caratteri a larghezza zero (*zero-width characters*) consentono di alterare la stringa grezza mantenendo intatto il significato semantico decodificato dall'LLM.

> [!WARNING]
> **Dato Empirico (2025):** Ricerche hanno dimostrato che le tecniche di evasione a livello di carattere hanno raggiunto un **tasso di bypass del 100%** contro molteplici guardrail commerciali di produzione, tra cui Microsoft Azure Prompt Shield.  
> Il limite strutturale dei filtri statici risiede nel fatto che **l'attaccante e l'LLM condividono un linguaggio semantico che la Regex non è in grado di decodificare**.

---

## 3. Classificatori Neurale AI (AI-Powered Classifiers)

Per superare i limiti delle corrispondenze letterali, si impiegano **classificatori neurali** dedicati all'analisi dell'intento malevolo (*intent classification*).

### Caso Studio: Meta Llama Prompt Guard 2
Un esempio di riferimento è **Llama Prompt Guard 2** di Meta (modelli compatti basati su architettura BERT da 22M e 86M parametri):
* **Funzionamento:** Analizza il testo in input e assegna un'etichetta probabilistica (`benigno` vs `malevolo/injection/jailbreak`).
* **Vantaggio:** Intercetta varianti inedite o prompt parafrasati perché addestrato su ampi dataset di injection noti.

### Limiti dei Classificatori Neurale
Anche i classificatori neurali possono essere aggirati: test di laboratorio hanno dimostrato che inserendo rumore semantico o lievi mutazioni di caratteri (circa il 30% di rumore), Prompt Guard 86M arriva a **classificare erroneamente come benigno il 98,2% dei prompt malevoli**.

---

## 4. Architettura a Cascata (Cascade Pattern)

L'introduzione di controlli di sicurezza impatta la latenza dell'applicazione. Negli ambienti di produzione, latenze superiori a **200 ms** degradano l'esperienza utente.

La best practice architetturale consiste nell'implementare un **pattern a cascata**, ordinato per costo computazionale crescente:

```
[ Input ] ──► [ 1. Regex / Blocklist ] ──► [ 2. Classificatore Neurale ] ──► [ 3. Inserimento nell'LLM ]
                     │ (Microsecondi)                  │ (Millisecondi)
                     └─► (Se fallisce: Drop)           └─► (Se fallisce: Drop)
```

1. **Step 1 - Controlli Rapidi (Regex / Regex-PII):** Bloccano attacchi opportunistici noti e ripuliscono i dati a costo computazionale zero ($\mu s$).
2. **Step 2 - Classificatore Neurale (Prompt Guard):** Esegue l'analisi semantica dell'intento solo sui payload che superano il primo filtro ($ms$).
3. **Step 3 - LLM-as-a-Judge (Solo per Flussi Asincroni/Critici):** L'uso di un secondo LLM per valutare la pericolosità è accurato ma richiede secondi ($s$), rendendolo inadatto a chat interattive in tempo reale.

---

## 5. Il Punto Cieco dei Guardrail: Indirect Prompt Injection

Un limite critico dei guardrail perimetrali tradizionali è che essi ispezionano prevalentemente l'input digitato dall'utente, ignorando i dati recuperati da fonti esterne (email, documenti aziendali, pagine web, knowledge base RAG).

* **Caso Reale (Vulnerabilità Slack AI 2024):** Ricercatori hanno dimostrato che caricando file contenenti comandi malevoli nascosti, Slack AI esfiltrava messaggi da canali privati a cui l'attaccante non aveva accesso. I guardrail sull'input utente non sono mai scattati poiché l'attacco è entrato attraverso il canale dei dati ingeriti.
* **Regola Difensiva:** Trattare tutti i blocchi RAG o testi estratti da API terze come **input non fidato**, sottoponendoli agli stessi guardrail di sanitizzazione prima dell'inserimento nella Context Window.

---

## 6. Deployment Controls: Gestione del Blast Radius & Minimo Privilegio

Anche con guardrail attivi, la postura di sicurezza deve assumere che l'iniezione possa avere successo. I **Deployment Controls** stabiliscono il raggio d'impatto (*blast radius*) dell'attacco:

```
┌────────────────────────────────────────────────────────┐
│                   INJECTION INGESTED                   │
└───────────────────────────┬────────────────────────────┘
                            │
            ┌───────────────┴───────────────┐
            ▼                               ▼
 [ Deployment Sovra-Privilegiato ]   [ Deployment Minimale (Least Privilege) ]
  • Accesso a tutti i DB aziendali    • Scoping basato sull'identità dell'utente
  • Permessi di esecuzione shell/eval • API read-only a portata circoscritta
  • Nessuna validazione output        • Validazione rigorosa degli schemi
            │                               │
            ▼                               ▼
    DATA BREACH / RCE               IMPATTO MINIMO / NO ACCESS
```

### Il Principio del Minimo Privilegio (Least Privilege)
* **Scoping nei Sistemi RAG:** Il motore vettoriale deve recuperare unicamente i documenti a cui l'utente autenticato ha diritto di accesso a livello di identità (RBAC). Se l'LLM legge da un bucket globale indiscriminato, l'injection esfiltra dati di qualsiasi reparto.
* **Restrizione degli Agenti AI:** Un agente AI che elabora dati esterni non deve possedere permessi di scrittura su filesystem, accesso a shell di sistema o capacità di inoltro e-mail senza conferma esplicita (**Human-in-the-Loop**).

---

## 7. OWASP LLM05: Improper Output Handling (Gestione Impropria dell'Output)

Un errore architetturale diffuso è considerare l'output del modello come un dato intrinsecamente fidato. Quando la risposta dell'LLM alimenta sistemi a valle, può trasformarsi in un payload malevolo:

```
┌──────────┐  Risposta Non Sanificata   ┌───────────────────────┐  Vulnerabilità Risultante
│ LLM Core │ ─────────────────────────► │ Sistema a Valle       │ ───────────────────────►
└──────────┘                            └───────────────────────┘
                                         • Browser Web            ──► Cross-Site Scripting (XSS)
                                         • Engine SQL             ──► SQL Injection (SQLi)
                                         • Shell di Sistema       ──► Remote Code Execution (RCE)
```

### Contromisure di Output Guardrail
1. **Sanitizzazione & Encoding:** Applicare l'HTML encoding sui testi restituiti prima del rendering nel browser web; utilizzare costrutti parametrizzati per le query a database.
2. **Schema Enforcement (Validazione degli Schemi):** Forzare l'output strutturato (es. JSON) tramite librerie di validazione tipizzata come **Pydantic** o JSON Schema prima di inoltrare i parametri alle API interne. Se il JSON non valida, il payload viene scartato.
3. **Divieto di Esecutori Dinamici:** Rimuovere l'accesso a funzioni interpreti dinamiche (`eval()`, `exec()`, runtime bash non isolati).

---

## 8. Telemetria Difensiva, Rate Limiting & Incident Detection

I controlli di monitoraggio consentono di individuare tempestivamente l'attacco e limitare i danni:

| Meccanismo di Controllo | Scopo Operativo | Indicatore di Compromissione (IoC) |
| :--- | :--- | :--- |
| **Rate Limiting** | Limita il consumo di risorse e restringe la finestra operativa dell'attaccante. | Picchi improvvisi nel consumo di token o sequenze ravvicinate di richieste. |
| **Centralized Logging** | Garantisce l'audit trail completo per l'analisi forense post-incidente. | Sequenze ripetute di prompt contenenti payload di override o tentativi di jailbreak. |
| **Behavioral Monitoring** | Rileva anomalie sintattiche o semantiche nelle risposte generate. | Output contenenti lunghezze anomale, chiavi API o blocchi di testo confidenziali. |

> **Conclusione:** Una difesa efficace per sistemi generativi si realizza integrando **System Prompt Hardening**, **Guardrail a Cascata**, **Minimo Privilegio a livello di Deployment** e **Sanitizzazione rigorosa dell'Output**.
