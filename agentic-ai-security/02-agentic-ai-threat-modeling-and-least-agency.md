# 🛡️ Agentic AI Threat Modeling, Least Agency & Component Security

* **Dominio:** Agentic AI Security, Identity & Access Management (IAM), Threat Modeling, Least Agency Governance
* **Standard di Riferimento:** OWASP Top 10 for LLMs (LLM06: Excessive Agency), OWASP Agentic AI Security Initiative (ASI01: Agent Goal Hijack, ASI04: Agentic Supply Chain), NIST SP 800-207 (Zero Trust Architecture)
* **Modulo TryHackMe:** Modulo 1 — Agentic AI Fundamentals (Stanza 2: How to Threat Model and Secure It)
* **Scopo:** Guida completa alla sicurezza degli ecosistemi agentici, analisi dell'escalation del rischio architetturale, applicazione pratica dei quattro pilastri del principio di **Least Agency** e matrice di threat modeling per tool, orchestratori e protocolli distribuiti.

---

## 1. L'Escalation del Rischio Architetturale nei Sistemi Agentic

L'evoluzione dai tradizionali modelli generativi (*text-in/text-out*) agli agenti autonomi operativi espande esponenzialmente la superficie d'attacco. Questa escalation si sviluppa attraverso 4 passaggi chiave:

```
┌────────────────────────────────────────────────────────────────────────┐
│               ESCALATION DEL RISCHIO NEI SISTEMI AGENTIC               │
├────────────────────────────────────────────────────────────────────────┤
│ 1. Rimozione dei Workflow Hardcoded                                    │
│    └─► I percorsi approvati dall'uomo spariscono; l'LLM decide le rotte│
├────────────────────────────────────────────────────────────────────────┤
│ 2. Esecuzione Reale tramite Tool                                       │
│    └─► Le decisioni probabilistiche diventano comandi su API, DB e OS  │
├────────────────────────────────────────────────────────────────────────┤
│ 3. Moltiplicazione Multi-Agent                                         │
│    └─► Più agenti collaborano, ognuno dotato di credenziali operative  │
├────────────────────────────────────────────────────────────────────────┤
│ 4. Estensione Esterna (MCP & A2A)                                      │
│    └─► L'agente interagisce con server e agenti esterni non fidati     │
└────────────────────────────────────────────────────────────────────────┘
```

### L'Assunzione Silenziosa e Fallace
Tutti e quattro i passaggi poggiano su una presupposizione implicita estremamente rischiosa: **che il livello di autorità e accesso accordato all'agente al momento della decisione sia esatto**. Se un agente riceve autorizzazioni eccedenti o non verificate, un singolo errore di pianificazione o un attacco di injection si traduce immediatamente in una violazione dell'infrastruttura reale.

---

## 2. Il Principio di Least Agency (OWASP)

Per colmare le lacune dei modelli di sicurezza tradizionali, **OWASP** ha formalizzato il principio di **Least Agency** (Minima Autonomia/Agenzia):

> *"Ampliamo i concetti di Least-Privilege ed Excessive Agency introducendo il principio di Least-Agency. Questo racchiude la nostra raccomandazione alle organizzazioni affinché evitino un'autonomia non necessaria; distribuire comportamenti agentici laddove non sono strettamente richiesti espande la superficie d'attacco senza aggiungere valore."* — **OWASP Agentic AI Security Initiative**

Il **Least Agency** rappresenta la sintesi tra il tradizionale **Principio del Minimo Privilegio (PoLP)** e la mitigazione del rischio **OWASP LLM06: Excessive Agency**.

### Differenza Strutturale: Least-Privilege Tradizionale vs Least Agency

Non si tratta di una semplice ridenominazione: il baricentro decisionale si sposta dall'umano al runtime probabilistico:

| Dimensione | Least-Privilege Tradizionale | Least Agency (Sistemi Agentic) |
| :--- | :--- | :--- |
| **Attore Decisionale** | **Umano** (amministratore/sviluppatore) che definisce ACL e ruoli. | **Modello Probabilistico (LLM)** che valuta e seleziona le azioni a runtime. |
| **Momento del Controllo** | **Statico / Pre-Deployment:** Approvato in fase di configurazione e verificato periodicamente. | **Dinamico / Just-in-Time:** L'agente decide sul momento quale risorsa attivare o verso chi autenticarsi. |
| **Natura del Flusso** | Deterministica, tracciata e riproducibile via codice. | Non deterministica, guidata dal contesto conversazionale e dal reasoning. |
| **Punto di Rottura** | Errore umano di configurazione permessi. | Manipolazione del contesto (*Prompt Injection*, *Goal Hijack*, allucinazioni). |

Poiché il processo decisionale si è trasferito all'LLM in tempo reale, i controlli di accesso statici non sono più sufficienti.

---

## 3. I Quattro Pilastri Pratici del Least Agency

Per applicare concretamente il Least Agency nella progettazione e governance degli agenti, occorre implementare quattro pilastri difensivi:

```
┌────────────────────────────────────────────────────────────────────────┐
│               I 4 PILASTRI PRATICI DEL LEAST AGENCY                    │
├────────────────────────────────────────────────────────────────────────┤
│ 1. Time-Boxed Access        ──► Permessi effimeri Just-In-Time (JIT)   │
│ 2. One Credential per Agent ──► Identità crittografiche isolate        │
│ 3. Tool-Bound Identity      ──► Privilegi vincolati a singole API      │
│ 4. Tracked Inheritance      ──► Attenuazione e audit della delega      │
└────────────────────────────────────────────────────────────────────────┘
```

### 1. Time-Boxed Access (Accesso Temporaneo, Non Permanente)
* **Regola:** Nessun agente deve detenere permessi permanenti (*standing access*).
* **Implementazione:** Emissione di token effimeri e credenziali JIT con validità limitata alla durata del singolo task. Una volta completata la chiamata, la credenziale decade automaticamente.

### 2. One Credential per Agent (Una Credenziale per Agente, Non Condivisa)
* **Regola:** È vietato trattare una pipeline multi-agent come un unico client con un unico token condiviso.
* **Implementazione:** Ogni worker dispone di una propria identità crittografica distinta (*Service Principal / Keypair*). Questo garantisce un audit trail puntuale e permette di revocare le autorizzazioni di un singolo agente compromesso senza bloccare l'intero sistema.

### 3. Tool-Bound Identity (Identità Vincolata al Singolo Tool)
* **Regola:** Evitare *service account* generici ad ampio raggio (es. `admin-role`).
* **Implementazione:** L'identità dell'agente deve mappare strettamente solo le funzioni o gli endpoint esplicitamente censiti nel toolset associato. Un agente addetto alle notifiche non deve avere accesso in scrittura su storage non pertinenti.

### 4. Tracked Inheritance (Ereditarietà Tracciata e Attenuata)
* **Regola:** La delega lungo la catena gerarchica non deve consentire privilege escalation silente.
* **Implementazione:** Quando l'orchestratore invoca un sotto-agente o un partner A2A, i privilegi trasmessi devono essere un **sottoinsieme stretto (*permission attenuation*)** di quelli del chiamante, e ogni passaggio di delega deve essere registrato in un registro immutabile.

---

## 4. Threat Modeling dei Componenti Agentici

La combinazione di capacità esecutive e protocolli di rete introduce vettori di minaccia specifici ad ogni livello dello stack:

```
┌────────────────────────────────────────────────────────────────────────┐
│               MATRICE DELLE MINACCE PER COMPONENTE AGENTIC             │
├────────────────────────────────────────────────────────────────────────┤
│ LIVELLO TOOL & ACI                                                     │
│   • Parameter / Command Injection via input non sanificati             │
│   • Tool Misuse per descrizioni ACI ambigue o ingannevoli              │
├────────────────────────────────────────────────────────────────────────┤
│ LIVELLO MULTI-AGENT & ORCHESTRAZIONE                                   │
│   • Cascading Failures / Loop infiniti di chiamate tra agenti          │
│   • Context Contamination / Propagazione di allucinazioni              │
├────────────────────────────────────────────────────────────────────────┤
│ LIVELLO PROTOCOLLI ESTERNI (MCP & A2A)                                 │
│   • Tool Poisoning tramite server MCP di terze parti non validati      │
│   • Indirect Prompt Injection nei dati restituiti dalle Resources      │
│   • Confused Deputy Attacks su deleghe trans-organizzative A2A         │
└────────────────────────────────────────────────────────────────────────┘
```

### Vettori Critici di Vulnerabilità:
1. **Tool Poisoning (MCP):** Un server MCP malevolo espone uno strumento la cui `description` contiene direttive di *Prompt Injection* invisibili all'utente. Quando l'Host elenca gli strumenti, il modello legge la descrizione malevola e devia dai suoi vincoli operativi.
2. **Indirect Prompt Injection da Risorse Dati:** Un tool esegue una query su una risorsa esterna (pagina web o documento). Il payload malevolo iniettato nel testo altera il ciclo di pianificazione successivo dell'agente.
3. **Confused Deputy su Deleghe A2A:** Un attaccante convince un agente autorizzato a richiedere a un agente partner esterno l'esecuzione di un'operazione privilegiata per cui l'utente originario non avrebbe avuto i permessi.

---

## 5. Mappatura OWASP Top 10 for Agentic AI (ASI) e Mitigazioni

Le minacce agentiche sono catalogate specificamente dall'iniziativa **OWASP Agentic AI**:

| Codice Minaccia | Nome | Descrizione & Meccanismo | Contromisura Architetturale |
| :--- | :--- | :--- | :--- |
| **ASI01** | **Agent Goal Hijack** | L'attaccante sovrascrive l'obiettivo originale dell'agente manipolando la memoria o il contesto. | Separazione rigida tra istruzioni di sistema e dati operativi; guardrail deterministici a runtime. |
| **ASI04** | **Agentic Supply Chain** | Compromissione di server MCP, librerie tool o Agent Card remote di terze parti. | *Allowlisting* crittografico dei server MCP; validazione di firma e schema delle Agent Card A2A. |
| **LLM06** | **Excessive Agency** | Concessione di eccessiva autonomia, funzionalità superflue o permessi non limitati. | Implementazione rigorosa dei **4 Pilastri di Least Agency**; *Human-in-the-Loop (HITL)* per azioni irreversibili. |

---

## 6. Checklist di Verifica e Monitoraggio

Prima di distribuire in produzione un'applicazione agentica, verificare:
* [ ] **Necessità dell'Autonomia:** È stato verificato che il task non possa essere risolto con un workflow deterministico convenzionale?
* [ ] **Validazione Rigida ACI:** Gli schemi dei parametri JSON sono vincolati con regex, tipi stretti e sanitizzazione backend?
* [ ] **Isolamento delle Credenziali:** Ciascun sotto-agente possiede la propria identità JIT distinta?
* [ ] **Allowlist MCP & A2A:** Le connessioni esterne sono limitate a endpoint verificati e certificati crittograficamente?
* [ ] **Audit Trail Continuo:** Tutte le invocazioni di tool (`tool_use` e `tool_result`), le decisioni di routing e le deleghe A2A sono loggate e monitorate contro pattern anomali?
