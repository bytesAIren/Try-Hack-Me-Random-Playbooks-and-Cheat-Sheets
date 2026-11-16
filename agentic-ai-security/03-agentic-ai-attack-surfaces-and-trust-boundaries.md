# 🎯 Agentic AI Attack Surfaces & Trust Boundaries

* **Dominio:** Agentic AI Security, Architectural Threat Modelling, Attack Surface Mapping, Trust Boundaries & Authority Governance
* **Standard di Riferimento:** OWASP Agentic AI Security Initiative (ASI01, ASI04), OWASP Top 10 for LLMs (LLM06: Excessive Agency), NIST SP 800-207 (Zero Trust Architecture)
* **Modulo TryHackMe:** Modulo 1 — Agentic AI Fundamentals (Stanza 2: Architectural Threat Modelling & Attack Surface)
* **Scopo:** Guida metodologica per eseguire il Threat Modelling su un'architettura multi-agent aziendale (caso di studio *TryTrainMe*), mappando componenti, blast radii, flussi di dati, confini di fiducia e la catena di propagazione che trasforma l'influenza informativa in autorità esecutiva.

---

## 1. Analisi Architetturale: L'Ecosistema Multi-Agent (Caso TryTrainMe)

Per modellare le minacce in un sistema agentico reale, analizziamo l'architettura introdotta da **TryTrainMe** per automatizzare i flussi operativi ingegneristici. Un **Orchestrator** centrale smista le richieste degli utenti verso agenti specializzati dotati di strumenti (*tool*) specifici:

```
┌────────────────────────────────────────────────────────────────────────┐
│               ARCHITETTURA MULTI-AGENT TRYTRAINME                      │
├────────────────────────────────────────────────────────────────────────┤
│                                                                        │
│                     ┌───────────────────────────┐                      │
│                     │   EMPLOYEE / INTERFACE    │                      │
│                     └─────────────┬─────────────┘                      │
│                                   │                                    │
│                                   ▼                                    │
│                     ┌───────────────────────────┐                      │
│                     │       ORCHESTRATOR        │                      │
│                     └─┬───┬───┬───┬───┬───┬───┬─┘                      │
│                       │   │   │   │   │   │   │                        │
│        ┌──────────────┘   │   │   │   │   │   └──────────────┐         │
│        ▼                  ▼   ▼   ▼   ▼   ▼                  ▼         │
│ ┌─────────────┐    ┌───────────────────────────┐      ┌─────────────┐  │
│ │ docs-agent  │    │ desk / access / deploy /  │      │ media-agent │  │
│ │ (Read-Only) │    │ mail / log agents         │      │ (Parsing)   │  │
│ └──────┬──────┘    └─────────────┬─────────────┘      └──────┬──────┘  │
│        │                         │                           │         │
│        ▼                         ▼                           ▼         │
│ [ Internal KB ]        [ Prod / IAM / Email ]        [ MCP / A2A Ext ] │
└────────────────────────────────────────────────────────────────────────┘
```

### Mappatura delle Capacità e dei Blast Radii

Nel Threat Modelling agentico il punto focale non è catalogare ogni singolo script, ma valutare il potenziale raggio d'impatto (**Blast Radius**) di ciascun componente in caso di compromissione:

| Agente | Ruolo Primario | Tool Esemplificativi | Livello di Rischio / Blast Radius |
| :--- | :--- | :--- | :--- |
| **`orchestrator`** | Coordina e delega i compiti | `queue_specialist_task` | **Critico:** Punto unico di smistamento, visibilità dell'intero contesto. |
| **`docs-agent`** | Recupera documentazione e runbook | `read_service`, `read_runbook` | **Basso/Medio:** Operazioni in sola lettura su basi di conoscenza interne. |
| **`desk-agent`** | Gestisce ticket e richieste di supporto | `read_request`, `update_request` | **Medio:** Modifica lo stato operativo dei record di assistenza. |
| **`access-agent`** | Assegna e revoca permessi IAM | `check_role`, `grant_access` | **Critico:** Può alterare i privilegi di accesso dell'intera organizzazione. |
| **`deploy-agent`** | Gestisce rilasci e pipeline CI/CD | `inspect_pipeline`, `deploy` | **Critico:** Può iniettare o interrompere carichi di lavoro in produzione. |
| **`mail-agent`** | Legge e trasmette comunicazioni email | `read_mail`, `send_mail` | **Alto:** Canale primario per esfiltrazione di dati riservati all'esterno. |
| **`log-agent`** | Consulta telemetria, eventi e log | `search_logs`, `read_metrics` | **Medio/Alto:** Esposizione potenziale di token, segreti e PII nei log. |
| **`media-agent`** | Converte e analizza audio/video/immagini | `transcribe_call`, `extract_text` | **Medio:** Vettore privilegiato di *Indirect Prompt Injection* multimodale. |

---

## 2. Identità Agentiche e Connessioni Esterne

### Identità Scoped vs Credenziali Condivise
Ogni invocazione di uno strumento viene eseguita sotto un'identità di sicurezza (**Security Principal**):
* **Anti-Pattern (Credenziale Amministrativa Condivisa):** Se tutti gli agenti operano con un unico service account generico (es. `shared-ai-admin`), la compromissione del `docs-agent` (in sola lettura) consente a un attaccante di ereditare permessi di deployment o di amministrazione IAM.
* **Best Practice (Scoped Identity / Least Agency):** Ogni agente deve possedere un'identità crittografica isolata (`docs-read-svc`, `deploy-prod-svc`, `mail-send-svc`), limitando rigidamente l'autorità effettiva al perimetro del proprio task.

### Connessioni Esterne (MCP e A2A)
TryTrainMe interagisce con server **MCP (Model Context Protocol)** e agenti partner esterni via **A2A (Agent-to-Agent)**. Ogni connessione esterna introduce un nuovo **Trust Boundary**, poiché istruzioni, contesti e risposte attraversano domini non controllati dall'organizzazione.

### Checklist Preliminare di Revisione Architetturale
Prima di modellare gli attacchi, verificare se:
1. Un agente può invocare tool non pertinenti al proprio ruolo dichiarato.
2. Un sotto-agente eredita silenziosamente le credenziali dell'orchestrator.
3. Dati esterni non fidati possono raggiungere la Context Window dell'agente.
4. Un server MCP esterno può alterare le decisioni operative del sistema.
5. Un agente può inviare istruzioni non verificate a un altro agente.
6. Esistono agenti in grado di produrre effetti visibili all'esterno dell'azienda.
7. Le azioni distruttive richiedono l'approvazione umana esplicita (*Human-in-the-Loop*).

---

## 3. I Quattro Pilastri del Threat Modelling per Sistemi Agentic

Una modellazione rigorosa analizza il sistema prima dell'attacco mappando **quattro elementi fondativi**:

```
┌────────────────────────────────────────────────────────────────────────┐
│               I 4 PILASTRI DEL THREAT MODELLING AGENTIC                │
├────────────────────────────────────────────────────────────────────────┤
│ 1. Assets           ──► Cosa ha valore (dati, credenziali, produzione) │
│ 2. Components       ──► Chi elabora i dati e a cosa è connesso         │
│ 3. Data Flows       ──► Come si muovono i dati (che diventano contesto)│
│ 4. Trust Boundaries ──► Dove cambia il livello di fiducia dei dati     │
└────────────────────────────────────────────────────────────────────────┘
```

1. **Assets (Beni da Proteggere):** Codice sorgente, credenziali CI/CD, database di produzione, record IAM, memoria degli agenti, dati personali dei clienti.
2. **Components (Componenti e Topologia):** Orchestrator, worker specializzati, memory store vettoriali, server MCP, tool esecutivi. Un componente va valutato per **ciò a cui è collegato** (un agente di lettura collegato anche a un agente mail diventa un ponte di esfiltrazione).
3. **Data Flows (Flussi di Dati Attivi):** Nei sistemi agentici i dati **non sono mai passivi**: documenti, email o risposte API finiscono nella Context Window e possono agire come istruzioni eseguibili.
4. **Trust Boundaries (Confini di Fiducia):** Si manifestano quando i dati passano tra entità con livelli di sicurezza differenti:

```
   [ UNTRUSTED DOMAIN ]                     [ TRUSTED DOMAIN ]
 External Web / Partner API ─────────────► Documentation Agent
                               ▲
                               │
                    TRUST BOUNDARY CROSSING
                  (I dati diventano Influenza)
```

> **Principio Chiave:** Nei sistemi AI autonomi, **i dati possono trasformarsi in influenza (*Data can become influence*)**. Leggere un dato esterno significa introdurlo nel processo decisionale dell'agente.

---

## 4. Dall'Influenza all'Autorità (From Influence to Authority)

La transizione più critica nel Threat Modelling agentico si verifica quando un'influenza informativa esterna incontra l'autorità esecutiva interna:

```
[ Dato Esterno Non Fidato ] ──► [ Ragionamento Agente ] ──► [ Invocazione deploy-prod() ]
        (Influenza)                  (Decisione)                    (Autorità)
```

L'**Autorità** è l'insieme di tool, credenziali e permessi che consentono all'agente di produrre effetti sul mondo reale. Essa è determinata da:
* I tool esposti all'agente.
* I privilegi del service principal associato a ciascun tool.
* La presenza o assenza di vincoli di convalida o gate *Human-in-the-Loop (HITL)*.

Se un agente esposto a dati esterni non fidati possiede un'autorità sproporzionata, un'iniezione di istruzioni si converte direttamente in un'azione malevola non autorizzata sull'infrastruttura.

---

## 5. Mappatura della Superficie d'Attacco e Catena di Propagazione

La **Superficie d'Attacco (*Attack Surface*)** agentica comprende ogni punto in cui un attaccante può introdurre dati, alterare lo stato o innescare funzionalità:

| Dimensione di Superficie | Meccanismo Operativo | Esempio nel Caso TryTrainMe |
| :--- | :--- | :--- |
| **Direct Inputs** | Prompt diretti inviati dall'utente all'interfaccia. | Prompt inviato da un dipendente all'Orchestrator. |
| **Indirect Inputs / Data** | Testo malevolo incorporato in sorgenti consultate dall'agente. | Istruzioni malevole nascoste in runbook, email o pagine web. |
| **Memory Store** | Iniezione di contesti malevoli persistenti (*Memory Poisoning*). | Istruzioni malevole salvate nel contesto condiviso dell'agente. |
| **Tools & Capabilities** | Funzionalità operative richiamabili a runtime. | Esecuzione di `deploy()`, modifiche DB o invio email. |
| **Credentials & IAM** | Privilegi e accessi garantiti agli strumenti. | Token API con permessi di scrittura eccessivi. |
| **Agent Communications** | Flussi di delega interna e protocolli MCP/A2A. | Chiamate cross-agent tra `desk-agent` e `access-agent`. |
| **Outputs & Effects** | Modifiche permanenti allo stato dell'ambiente. | Rilascio di codice in produzione, cancellazione ticket. |

### La Catena di Propagazione Sistemica
Queste superfici non operano in modo isolato: l'attacco si propaga concatenando i diversi layer architetturali:

```
┌────────────────────────────────────────────────────────────────────────┐
│           CATENA DI PROPAGAZIONE SULLA SUPERFICIE D'ATTACCO            │
├────────────────────────────────────────────────────────────────────────┤
│ [ Indirect Input ] ──► [ Agent Memory ] ──► [ Privileged Tool ]        │
│    (Ingresso)            (Persistenza)          (Autorità)             │
│                                                     │                  │
│                                                     ▼                  │
│                                          [ MCP / A2A / Other Agents ]  │
│                                                 (Propagazione)         │
└────────────────────────────────────────────────────────────────────────┘
```

---

## 6. Distinzione Metodologica: Attack Surface vs Vulnerability

Nel Threat Modelling è essenziale distinguere formalmente tra capacità esposta e difetto di sicurezza:

* **Superficie d'Attacco (Capacità Legittima):** Identifica *cosa può essere raggiunto o invocato*. Che `mail-agent` disponga del tool `send_mail()` è una caratteristica funzionale voluta, non un bug.
* **Vulnerabilità (Mancanza di Controlli):** Rappresenta l'assenza di salvaguardie adeguate a prevenire l'abuso della capacità. Permettere a `send_mail()` di inviare messaggi a destinatari arbitrari esterni senza restrizioni di dominio o conferma umana trasforma la capacità in una vulnerabilità di esfiltrazione dati.

```
┌────────────────────────────────────────────────────────────────────────┐
│               DISTINZIONE METODOLOGICA NEL THREAT MODELLING            │
├────────────────────────────────────────────────────────────────────────┤
│ 1. Identificare l'Attack Surface ──► "mail-agent può usare send_mail()"│
│ 2. Valutare i Controlli Esistenti ──► "Può inviare all'esterno senza   │
│                                        approvazione HITL o filtri?"    │
│ 3. Determinare la Vulnerability  ──► "Vulnerabilità di esfiltrazione   │
│                                        tramite Indirect Prompt Inject" │
└────────────────────────────────────────────────────────────────────────┘
```

---

## 7. Le Cinque Domande Guida del Threat Modeller

Ogni connessione o confine di fiducia tracciato nel Data Flow Diagram (DFD) agentico deve essere interrogato attraverso cinque domande sistematiche:

| # | Domanda Guida | Obiettivo di Analisi di Sicurezza |
| :--- | :--- | :--- |
| **1** | **Cosa attraversa questo confine?** | Identificare il tipo di payload (istruzione utente, documento RAG, schema JSON, risposta MCP). |
| **2** | **Chi controlla realmente quel dato?** | Verificare l'origine del dato (utente interno, fornitore terzo, utente esterno anonimo). |
| **3** | **Perché il ricevente si fida del dato?** | Individuare assunzioni implicite (mancata sanitizzazione, assenza di delimitatori strutturati). |
| **4** | **Quale autorità diventa raggiungibile?** | Mappare tutti i tool e i sistemi a valle che l'agente può attivare dopo l'elaborazione. |
| **5** | **Cosa accade se l'assunzione fallisce?** | Calcolare il peggior scenario d'impatto (*worst-case blast radius*) su riservatezza, integrità e disponibilità. |
