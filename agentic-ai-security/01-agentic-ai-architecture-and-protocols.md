# 🤖 Agentic AI Architecture, Patterns & Communication Protocols

* **Dominio:** Agentic AI Architecture, Autonomous Systems, LLM Orchestration & Inter-Agent Protocols
* **Standard di Riferimento:** ReAct Paradigm (Yao et al., 2023), Model Context Protocol (MCP Specification), Agent-to-Agent Protocol (A2A Specification), OWASP Agentic AI Security Initiative
* **Modulo TryHackMe:** Modulo 1 — Agentic AI Fundamentals (Stanza 1: How Agentic AI Works)
* **Scopo:** Guida tecnica e architetturale all'evoluzione dai modelli LLM tradizionali ai sistemi agentici autonomi, analisi del ciclo operativo ReAct e dei tool, pattern di orchestrazione Multi-Agent e protocolli di comunicazione esterna standardizzati (MCP e A2A).

---

## 1. Cosa Rende "Agentic" un Sistema AI (The Agentic Loop)

Secondo la definizione formale adottata da **OWASP**, un agente AI è un sistema che **"percepisce, ragiona, decide e agisce autonomamente per raggiungere un obiettivo"**. Questo ciclo operativo continuo trasforma un generatore probabilistico di testo in un attore dotato di reale autonomia operativa:

```
┌────────────────────────────────────────────────────────────────────────┐
│                          THE AGENTIC LOOP                              │
├────────────────────────────────────────────────────────────────────────┤
│                                                                        │
│   ┌──────────────┐      ┌──────────────┐      ┌──────────────┐         │
│   │ 1. PERCEIVE  │ ───► │  2. REASON   │ ───► │  3. DECIDE   │         │
│   │  (Input/Dati)│      │  (Pianifica) │      │ (Sceglie Tool)│        │
│   └──────────────┘      └──────────────┘      └───────┬──────┘         │
│          ▲                                            │                │
│          │              ┌──────────────┐              │                │
│          └───────────── │    4. ACT    │ ◄────────────┘                │
│         (Feedback/Stato)│ (Esegue Azione)│                             │
│                         └──────────────┘                               │
└────────────────────────────────────────────────────────────────────────┘
```

### L'Evoluzione Architetturale in 3 Stadi

Per comprendere dove si colloca esattamente il confine dell'autonomia (*agency*), è utile osservare l'evoluzione di un assistente AI attraverso tre stadi architetturali:

1. **Stadio 1: Vanilla LLM (Assistente Conversazionale Base)**
   * Il modello attinge solo ai pesi pre-addestrati. Privo di accesso a fonti esterne o strumenti, la transazione si esaurisce nello scambio testuale.
2. **Stadio 2: Workflow Predefiniti (Automazione Hardcoded)**
   * Il sistema dispone di tool o sotto-agenti, ma segue percorsi rigidi (*predefined routes*, `if/else`). La logica è deterministica e cablata dal programmatore: non è ancora un sistema agentic, e possiede un limite strutturale invalicabile (*ceiling*).
3. **Stadio 3: Vera Autonomia (Agentic System)**
   * I percorsi hardcoded vengono rimossi. Al modello viene fornito un obiettivo e l'elenco delle risorse disponibili (strumenti e sotto-agenti); l'agente ragiona e decide a runtime quale tool invocare, in quale sequenza e come correggere il tiro in base al feedback.

| Caratteristica | Vanilla LLM | Workflow Hardcoded | Sistema Agentic Autonomo |
| :--- | :--- | :--- | :--- |
| **Accesso Esterno** | Nessuno (solo memoria interna). | Regole statiche preimpostate. | Dinamico, deciso a runtime dall'LLM. |
| **Logica Decisionale** | Assente (generazione testuale). | Deterministica / Cablata nel codice. | Autonoma, guidata dal ragionamento. |
| **Flessibilità** | Vincolata al prompt corrente. | Limitata ai percorsi previsti dallo sviluppatore. | Elevata: adatta il piano all'obiettivo. |
| **Superficie di Rischio** | Allucinazioni, Prompt Injection. | Errori di logica o bypass delle rotte. | **Excessive Agency (OWASP LLM06)**, abusi di tool. |

> **Principio Architetturale:** La differenza tra un semplice LLM e un sistema agentic non risiede nel modello sottostante, ma **nell'architettura che lo circonda** e nella libertà decisionale concessa al runtime.

---

## 2. Le Tre Capacità Fondamentali di un'Architettura Agentic (OWASP)

Affinché l'*Agentic Loop* operi in modo autosufficiente, l'infrastruttura attorno al modello deve fornire tre capacità strutturali:

```
┌────────────────────────────────────────────────────────────────────────┐
│               CAPACITÀ FONDAMENTALI DI UN SISTEMA AGENTIC              │
├────────────────────────────────────────────────────────────────────────┤
│ 1. Planning & Reasoning                                                │
│    • Scompone l'obiettivo in sotto-task sequenziali                    │
│    • Determina l'ordine logico e si auto-corregge se un passo fallisce │
├────────────────────────────────────────────────────────────────────────┤
│ 2. Memory & Statefulness                                               │
│    • Short-Term Memory: mantiene il contesto nella finestra di dialogo │
│    • Long-Term Memory: archivio esterno persistente (Vector DB / SQL)  │
├────────────────────────────────────────────────────────────────────────┤
│ 3. Action & Tool Use                                                   │
│    • Interagisce con API, database, file system e servizi esterni      │
│    • Trasforma una decisione astratta in un'azione concreta nel mondo  │
└────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Il Motore Operativo: ReAct Framework e Grounding

Il paradigma fondativo dell'Agentic Loop è la tecnica accademica **ReAct (Reasoning + Acting)** (Yao et al., 2023). Essa supera i due difetti intrinseci dei modelli linguistici isolati:
* **Ungrounded Reasoning:** Il puro ragionamento senza verifica empirica produce allucinazioni.
* **Unreasoned Acting:** La pura esecuzione di comandi priva di riflessione non consente di interpretare errori e adattare la strategia.

```
┌────────────────────────────────────────────────────────────────────────┐
│                    IL CICLO DI GROUNDING (ReAct)                       │
├────────────────────────────────────────────────────────────────────────┤
│                                                                        │
│   ┌───────────────────────┐               ┌───────────────────────┐    │
│   │       REASONING       │ ────────────► │        ACTION         │    │
│   │  (Formula ipotesi e   │               │  (Invoca Tool / Esegue│    │
│   │   pianifica il passo) │               │   codice nell'ambiente│    │
│   └───────────▲───────────┘               └───────────┬───────────┘    │
│               │                                       │                │
│               │         ┌───────────────────┐         │                │
│               └──────── │   GROUND TRUTH    │ ◄───────┘                │
│                         │  (Feedback reale  │                          │
│                         │   dall'output)    │                          │
│                         └───────────────────┘                          │
└────────────────────────────────────────────────────────────────────────┘
```

Alternando ragionamento e azione, l'output dello strumento fornisce **Ground Truth** oggettiva che aggiorna il contesto dell'agente, guidando il passo di ragionamento successivo.

---

## 4. Anatomia e Meccanica di Esecuzione dei Tool

Un **Tool** è un'interfaccia formale che consente all'agente di interagire con il mondo esterno.

> **Regola Fondamentale:** L'LLM **non esegue codice direttamente**. Il modello produce esclusivamente una stringa o payload strutturato (tipicamente JSON). È l'applicazione ospitante (*Agent Runtime*) che valida la richiesta, esegue il codice nel backend e restituisce il risultato al modello.

Ogni strumento è definito da 3 elementi:
1. `name`: Identificatore univoco della funzione.
2. `description`: Scopo del tool e contesto semantico in cui deve essere attivato.
3. `input_schema`: Schema tipizzato (JSON Schema) dei parametri richiesti.

### Il Ciclo di Esecuzione (Esempio Pratico)

#### 1. Definizione (Tool Definition fornita all'LLM)
```json
{
  "name": "check_role",
  "description": "Look up an employee's role and access permissions by employee ID.",
  "input_schema": {
    "type": "object",
    "properties": {
      "employee_id": {
        "type": "string",
        "description": "The employee's ID (es. e-4471)"
      }
    },
    "required": ["employee_id"]
  }
}
```

#### 2. Richiesta di Utilizzo (Tool Use Request emessa dall'LLM)
```json
{
  "type": "tool_use",
  "id": "toolu_01A3B",
  "name": "check_role",
  "input": {
    "employee_id": "e-4471"
  }
}
```

#### 3. Restituzione del Risultato (Tool Result inviato dal Runtime all'LLM)
```json
{
  "type": "tool_result",
  "tool_use_id": "toolu_01A3B",
  "content": "role: ml-engineer (contract); permitted_scopes: [training-data-buckets: read]"
}
```

### Principi di Progettazione ACI (Agent-Computer Interface)
Così come l'interfaccia utente (HCI) guida gli umani, l'**ACI** governa l'accuratezza con cui l'agente interagisce con le API:
* **Descrizioni Univoche e Non Ambigue:** Evitano che il modello selezioni funzioni distruttive per compiti ordinari.
* **Schemi Rigidi e Tipizzati:** Riducono il rischio di iniezione di payload malevoli nei parametri (*Parameter/SQL/Command Injection*).
* **Granularità Atomica:** Favorisce il principio del minimo privilegio, evitando tool onnipotenti.
* **Error Handling Informativo:** Messaggi di errore strutturati che consentono all'agente di auto-correggersi senza esporre stack trace interni.

---

## 5. Sistemi Multi-Agent: Topologia e Meccanica della Delega

Affidare l'intero carico a un singolo agente monolitico comporta rischi di allucinazione, saturazione della finestra di contesto e superfici di privilegio eccessive. L'architettura **Multi-Agent** suddivide il carico tra entità specializzate.

| Vantaggio Architetturale | Meccanismo Tecnico | Impatto Operativo |
| :--- | :--- | :--- |
| **Scoped Identity & Blast Radius** | Ogni agente ha credenziali isolate minime. | Se un sotto-agente è compromesso, il danno resta confinato. |
| **Reduced Path Dependency** | Ogni worker parte con un contesto pulito (*fresh context*). | Un'allucinazione o errore di un worker non contamina l'orchestratore. |
| **Specialisation** | Prompt di sistema focalizzati e toolset ristretti. | Maggiore accuratezza e affidabilità rispetto a un modello generalista. |
| **Speed (Parallelizzazione)** | Task indipendenti eseguiti simultaneamente. | Abbattimento della latenza complessiva di esecuzione. |

*Trade-off operativi:* Consumo di token fino a **15 volte superiore**, complessità di sincronizzazione dello stato e difficoltà di debugging distribuito.

### Il Pattern Orchestrator-Workers (OWASP Coordinating Agent)

La topologia predominante è gerarchica: un coordinatore riceve l'obiettivo, scompone i task, li assegna a worker specializzati e ne sintetizza l'output:

```
┌────────────────────────────────────────────────────────────────────────┐
│                 TOPOLOGIA ORCHESTRATOR-WORKERS (OWASP)                 │
├────────────────────────────────────────────────────────────────────────┤
│                                                                        │
│                     ┌───────────────────────────┐                      │
│                     │    ORCHESTRATOR AGENT     │                      │
│                     │   (Coordinating Agent)    │                      │
│                     └──────┬─────────────┬──────┘                      │
│                            │             │                             │
│              Delega Task A │             │ Delega Task B               │
│                            ▼             ▼                             │
│                ┌─────────────────┐ ┌─────────────────┐                 │
│                │  WORKER AGENT 1 │ │  WORKER AGENT 2 │                 │
│                │ (es. Refund)    │ │ (es. Calendar)  │                 │
│                └────────┬────────┘ └────────┬────────┘                 │
│                         │                   │                          │
│                         └─────────┬─────────┘                          │
│                                   ▼                                    │
│                   Riportano esclusivamente all'Orchestrator            │
│                   (Nessuna comunicazione diretta tra Worker)           │
└────────────────────────────────────────────────────────────────────────┘
```

### Meccanica Tecnica della Delega: Sotto-Agenti come Tool (*Tool Wrapping*)
Per l'orchestratore, delegare a un sotto-agente non richiede un protocollo speciale: **utilizza esattamente lo stesso ciclo `tool_use` / `tool_result`**.
* **OpenAI SDK (Handoffs):** Il passaggio a un agente specializzato viene esposto come un tool (es. `transfer_to_refund_agent`).
* **LangChain (Supervisor):** L'agente supervisore invoca il worker incapsulato come fosse una normale funzione (`schedule_event`).

---

## 6. Comunicazione Esterna: Protocolli MCP e A2A

Quando l'architettura supera il perimetro della singola applicazione locale (*in-process*), l'integrazione richiede protocolli standard aperti:

```
┌────────────────────────────────────────────────────────────────────────┐
│               ECOSISTEMA DI COMUNICAZIONE ESTERNA AGENTIC              │
├────────────────────────────────────────────────────────────────────────┤
│                                                                        │
│                      ┌─────────────────────────┐                       │
│                      │   AGENTIC APPLICATION   │                       │
│                      └────┬───────────────┬────┘                       │
│                           │               │                            │
│              Protocollo MCP               Protocollo A2A               │
│            (Agent-to-Tool)              (Agent-to-Agent)               │
│                           ▼               ▼                            │
│                ┌──────────────────┐ ┌──────────────────┐               │
│                │   MCP SERVER     │ │  EXTERNAL AGENT  │               │
│                │ (DB, API, File)  │ │ (Terze Parti)    │               │
│                └──────────────────┘ └──────────────────┘               │
└────────────────────────────────────────────────────────────────────────┘
```

### Model Context Protocol (MCP): La "Porta USB" per i Tool
Sviluppato per evitare connettori proprietari ad-hoc, MCP connette un'applicazione AI a risorse e strumenti esterni:
* **Host:** L'applicazione AI che ospita il modello e coordina il workflow (es. IDE, runtime).
* **Client:** Il componente interno all'Host che mantiene la connessione 1:1 con il server.
* **Server:** Il servizio esterno che espone **Tools** (funzioni eseguibili), **Resources** (dati consultabili) e **Prompts** (template predefiniti).

### Agent-to-Agent Protocol (A2A): Cooperazione Cross-Organization
A2A disciplina la cooperazione quando un agente deve delegare compiti a un agente autonomo remoto gestito da terze parti:

```
[ Client Agent ]                                          [ Remote Partner Agent ]
      │                                                              │
      ├─── 1. DISCOVERY (HTTP GET /.well-known/agent.json) ─────────►│
      │◄────── Restituisce "Agent Card" (Capacità & Auth Specs) ─────┤
      │                                                              │
      ├─── 2. AUTHENTICATION (OAuth2 / API Key / mTLS) ─────────────►│
      │◄────── Token / Sessione Autenticata ─────────────────────────┤
      │                                                              │
      ├─── 3. sendMessage (Task formattato secondo Card) ───────────►│
      │        [ L'agente remoto ragiona ed esegue il suo ciclo ]    │
      │◄────── Risposta strutturata finale ──────────────────────────┤
      │                                                              │
      ├─── 4. sendMessageStream (Opzionale per task prolungati) ────►│
      │◄══════ Aggiornamenti intermedi in streaming ═════════════════┤
```

### Tabella Sinottica: MCP vs A2A

| Dimensione | Model Context Protocol (MCP) | Agent-to-Agent Protocol (A2A) |
| :--- | :--- | :--- |
| **Interfaccia** | **Agent-to-Tool** (Agente verso Funzione/Dati). | **Agent-to-Agent** (Agente verso Agente Autonomo). |
| **Scopo Primario** | Esporre database, file e API esterne come tool standardizzati. | Scoperta, autenticazione e delega verso agenti di terze parti. |
| **Natura del Target** | Deterministica / Passiva (esegue codice o legge dati). | Autonoma / Attiva (possiede un proprio ciclo ReAct e usa MCP a sua volta). |
| **Flusso Combinato** | Un agente usa **A2A** per ingaggiare un partner esterno; quest'ultimo usa **MCP** per interrogare i propri DB interni. |
