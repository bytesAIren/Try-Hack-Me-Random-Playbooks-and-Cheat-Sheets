# 🦾 Agentic AI Security: Architecture, Threat Modeling & Defense Playbooks

* **Dominio:** Agentic AI Security, Autonomous Systems Architecture, Multi-Agent Orchestration, Inter-Agent Protocols & Least Agency Governance
* **Standard di Riferimento:** OWASP Agentic AI Security Initiative, OWASP Top 10 for LLMs (LLM06: Excessive Agency), Model Context Protocol (MCP), Agent-to-Agent Protocol (A2A), NIST SP 800-207 (Zero Trust)
* **Percorso Formativo:** TryHackMe — **Agentic AI Security Path** (5 Moduli tematici)
* **Scopo:** Knowledge base specialistica, prontuario architetturale e cheat sheet operativo per comprendere il funzionamento, condurre threat modeling e implementare controlli difensivi su sistemi ad agenti autonomi, ambienti multi-agent e protocolli distribuiti.

---

## 🧭 Mappa del Percorso Didattico (I 5 Moduli TryHackMe)

Il percorso sulla sicurezza dei sistemi Agentic è strutturato in **5 Moduli logici progressivi**:

```
[ Modulo 1: Fundamentals & Protocols ] ──► [ Modulo 2: Agent Hijacking ] ──► [ Modulo 3: Components & Authority ] ──► [ Modulo 4: Runtime & Memory ] ──► [ Modulo 5: Rogue Agents & Systemic Failure ]
     (How it works & Threat Modeling)          (Goal Redirection & Defense)            (Tools, Identity & IAM)               (Execution, State & Comms)             (Containment & Escalation Response)
```

---

## 📚 Indice Dettagliato dei Moduli e delle Note

### 🧩 MODULO 1: AGENTIC AI FUNDAMENTALS
*Comprendere come operano gli agenti autonomi, come mappare la superficie di rischio e come applicare la governance di sicurezza.*

* [**`01-agentic-ai-architecture-and-protocols.md`**](01-agentic-ai-architecture-and-protocols.md)  
  *Architettura Agentic, Framework ReAct, Pattern Multi-Agent e Protocolli MCP/A2A (How Agentic AI Works).*  
  * **The Agentic Loop:** Definizione OWASP (*Perceive $\rightarrow$ Reason $\rightarrow$ Decide $\rightarrow$ Act*), transizione evolutiva da Vanilla LLM a Workflow Hardcoded fino all'autonomia decisionale a runtime.
  * **Le 3 Capacità Fondamentali:** Planning & Reasoning, Memory & Statefulness, Action & Tool Use.
  * **Motore ReAct e Grounding:** Risoluzione di *ungrounded reasoning* e *unreasoned acting* attraverso il ciclo iterativo con la *ground truth*.
  * **Anatomia dei Tool & ACI:** Schema JSON (`tool_use` / `tool_result`), disaccoppiamento tra LLM e runtime applicativo, linee guida di progettazione per l'**Agent-Computer Interface (ACI)**.
  * **Sistemi Multi-Agent:** Contenimento del *blast radius*, contesti puliti, parallelizzazione, pattern gerarchico *Orchestrator-Workers* (OWASP Coordinating Agent) e tecnica del *Tool Wrapping/Handoffs*.
  * **Protocolli Esterni:** Specifiche aperte **Model Context Protocol (MCP)** (*Agent-to-Tool*, ruoli Host/Client/Server) e **Agent-to-Agent (A2A)** (*Discovery, Agent Card, Auth, sendMessage, streaming*).

* [**`02-agentic-ai-threat-modeling-and-least-agency.md`**](02-agentic-ai-threat-modeling-and-least-agency.md)  
  *Threat Modeling, Escalation del Rischio, Principio di Least Agency e Mitigazioni OWASP (How to Secure It).*  
  * **Escalation della Superficie d'Attacco:** I 4 stadi di esposizione e confutazione dell'assunzione silenziosa sull'autonomia del modello a runtime.
  * **Il Principio di Least Agency (OWASP):** Superamento del Least-Privilege tradizionale (umano, statico, pre-deployment) verso un modello dinamico just-in-time per LLM probabilistici; mitigazione di **OWASP LLM06: Excessive Agency**.
  * **I 4 Pilastri Pratici di Governance IAM:**
    1. *Time-Boxed Access:* Credenziali effimere Just-In-Time (JIT) a scadenza automatica.
    2. *One Credential per Agent:* Identità crittografiche isolate, divieto di token condivisi nella pipeline.
    3. *Tool-Bound Identity:* Privilegi granulari vincolati a singole API atomiche.
    4. *Tracked Inheritance:* Tracciamento e attenuazione della delega (*permission attenuation*) senza privilege escalation silente.
  * **Threat Modeling dei Componenti:** Vettori di attacco su ACI (parameter injection), Multi-Agent (propagazione a cascata, contaminazione del contesto) e protocolli esterni (Tool Poisoning su MCP, Indirect Prompt Injection, Confused Deputy su A2A).
  * **Mappatura OWASP Agentic AI:** Riferimento formale a **ASI01** (*Agent Goal Hijack*), **ASI04** (*Agentic Supply Chain*) e checklist pre-deployment.

* [**`03-agentic-ai-attack-surfaces-and-trust-boundaries.md`**](03-agentic-ai-attack-surfaces-and-trust-boundaries.md)  
  *Mappatura Architetturale, Superfici d'Attacco, Confini di Fiducia e Autorità (Caso TryTrainMe).*  
  * **Ecosistema Multi-Agent TryTrainMe:** Topologia dell'Orchestrator, agenti specializzati e analisi comparativa dei *Blast Radii* (da `docs-agent` a sola lettura fino a `deploy-agent` e `access-agent` critici).
  * **Identità Scoped vs Credenziali Condivise:** Isolamento delle credenziali e impatto dei protocolli di rete (MCP e A2A) come vettori cross-boundary.
  * **I 4 Pilastri del Threat Modelling Agentic:** Assets, Components, Data Flows e Trust Boundaries (*Data can become influence*).
  * **Dall'Influenza all'Autorità:** Transizione critica da contenuto informativo non fidato a esecuzione privilegiata di tool sull'infrastruttura.
  * **Catena di Propagazione e Superfici:** Mappatura su 6 dimensioni operative (Inputs, External Data, Memory, Tools, Credentials, Agent Communication) e catena sistemica (Ingresso $\rightarrow$ Persistenza $\rightarrow$ Autorità $\rightarrow$ Propagazione).
  * **Superficie d'Attacco vs Vulnerabilità:** Distinzione formale tra capacità legittima esposta e assenza di controlli; le 5 domande guida del threat modeller.

* [**`04-agentic-ai-threat-classification-stride-and-prioritization.md`**](04-agentic-ai-threat-classification-stride-and-prioritization.md)  
  *Classificazione STRIDE per Agenti, Framework OWASP, Prioritizzazione del Rischio e Security Findings.*  
  * **Le 3 Prospettive Complementari:** Convergenza tra STRIDE (proprietà violata), OWASP Threats & Mitigations (architettura abilitante) e OWASP Agentic Top 10 (pattern industriale ricorrente).
  * **Matrice STRIDE per Sistemi Agentic:** Declinazione operativa di Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service (Denial of Wallet) ed Elevation of Privilege.
  * **OWASP Agentic Threats su 7 Aree:** Inputs, Memory, Tools, Credentials, Delegation, MCP/External Services e Outputs/Effects.
  * **Caso Studio Integrato (Manipolazione Ticket TryTrainMe):** Scomposizione analitica completa del percorso d'attacco su `desk-agent` attraverso i tre framework.
  * **I 5 Fattori di Prioritizzazione:** Reachability, Authority, Blast Radius, Detectability e Reproducibility.
  * **Standard di Documentazione:** Template ingegneristico per redigere *Security Finding* azionabili (Attack Path, Agentic Weakness, Security Impact, Priority & Remediation).

---

### 🎯 MODULO 2: AGENT HIJACKING *(In Lavorazione)*
*Studio delle tecniche con cui gli attaccanti deviano l'obiettivo originale dell'agente e delle relative strategie di rilevamento e blocco.*
* *Focus:* Goal Hijacking (ASI01), Prompt Injection multi-turno mirata al piano di esecuzione, manipolazione delle istruzioni di sistema a runtime e tecniche di resilienza dell'orchestratore.

---

### 🔑 MODULO 3: AGENT COMPONENTS & AUTHORITY *(In Lavorazione)*
*Analisi degli abusi su strumenti, credenziali e componenti da cui gli agenti dipendono, e strategie di hardening dell'autorità.*
* *Focus:* Abuso di Service Account, privilege escalation orizzontale/verticale tra agenti, spoofing dell'identità crittografica e policy enforcement deterministico sulle chiamate ai tool.

---

### 🧠 MODULO 4: AGENT RUNTIME & MEMORY *(In Lavorazione)*
*Vulnerabilità a livello di esecuzione, persistenza della memoria e canali di comunicazione inter-agente.*
* *Focus:* Memory Poisoning (Vector Store & Short-Term Memory injection), manipolazione dei messaggi nel bus inter-agente, data exfiltration attraverso side-channel di streaming e isolamento dell'ambiente di runtime.

---

### 🛑 MODULO 5: ROGUE AGENTS & SYSTEMIC FAILURE *(In Lavorazione)*
*Propagazione ed escalation dei fallimenti sistemici negli ecosistemi agentici e meccanismi di contenimento e risposta.*
* *Focus:* Cicli infiniti di chiamata con esaurimento risorse (Denial of Wallet), cascading failure tra swarm di agenti, kill switch deterministici, circuit breakers e containment playbook per incident response.

---

## 🛡️ Principi Fondamentali di Difesa

> [!IMPORTANT]
> ### I Tre Comandamenti della Sicurezza Agentica
> 1. **Evitare l'Autonomia Non Necessaria (Least Agency):** Se un flusso decisionale può essere gestito tramite codice deterministico (`if/else`), non implementarlo come agente autonomo.
> 2. **Zero Standing Privileges:** Nessun agente deve possedere credenziali permanenti; applicare sempre identità effimere vincolate al singolo tool e time-boxed.
> 3. **Human-in-the-Loop (HITL) per Azioni Critiche:** Le operazioni distruttive, finanziarie o irreversibili devono richiedere una conferma esplicita out-of-band da parte di un operatore umano prima che il runtime esegua il `tool_result`.
