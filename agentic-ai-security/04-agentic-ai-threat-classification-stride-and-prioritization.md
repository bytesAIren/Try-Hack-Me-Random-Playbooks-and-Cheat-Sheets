# 📊 Agentic AI Threat Classification, STRIDE & Risk Prioritization

* **Dominio:** Agentic AI Security, Threat Classification Frameworks, Risk Prioritisation, Security Finding Documentation
* **Standard di Riferimento:** Microsoft STRIDE for AI, OWASP Agentic AI – Threats & Mitigations, OWASP Top 10 for Agentic Applications (ASI01–ASI10), CVSS for AI Systems
* **Modulo TryHackMe:** Modulo 1 — Agentic AI Fundamentals (Stanza 3: Threat Classification & Risk Prioritization)
* **Scopo:** Guida metodologica per identificare, classificare e prioritizzare le minacce nei sistemi multi-agent combinando tre prospettive complementari (STRIDE, architettura agentica OWASP e OWASP Agentic Top 10), con disamina del caso studio pratico di manipolazione dei ticket e standard per la stesura di *Security Finding* operativi.

---

## 1. Un Linguaggio Comune per Descrivere le Minacce Agentic

Una volta mappata l'architettura, individuati i confini di fiducia e tracciate le superfici d'attacco, è indispensabile adottare un linguaggio comune e strutturato per formalizzare le minacce. La descrizione di una minaccia agentica richiede la convergenza di **tre prospettive analitiche complementari**:

```
┌────────────────────────────────────────────────────────────────────────┐
│               LE 3 PROSPETTIVE COMPLEMENTARI DELLE MINACCE             │
├────────────────────────────────────────────────────────────────────────┤
│ 1. STRIDE                       ──► Quale proprietà di sicurezza cade? │
│ 2. OWASP Threats & Mitigations  ──► Quale componente abilita l'abuso?  │
│ 3. OWASP Agentic Top 10         ──► Quale classe di rischio affrontiamo?│
└────────────────────────────────────────────────────────────────────────┘
```

| Prospettiva / Framework | Domanda Guida dell'Analisi | Obiettivo Metodologico |
| :--- | :--- | :--- |
| **Microsoft STRIDE** | *Quale proprietà fondamentale di sicurezza è stata violata?* | Isola l'impatto tecnico primario (riservatezza, integrità, disponibilità, autenticità). |
| **OWASP Agentic Threats & Mitigations** | *In quale modo la specifica architettura agentica ha abilitato l'attacco?* | Analizza le assunzioni di fiducia fallite tra memoria, tool, deleghe, identità e connettori MCP/A2A. |
| **OWASP Agentic Top 10 (ASI01–ASI10)** | *A quale pattern di rischio ricorrente su scala industriale corrisponde il problema?* | Associa la specifica falla a categorie standardizzate per facilitare la prioritizzazione del remediation. |

---

## 2. STRIDE per Sistemi Agentic: Analisi delle Violazioni di Proprietà

Il modello **STRIDE** di Microsoft, originariamente progettato per il software convenzionale, si applica ai sistemi autonomi estendendo ogni categoria alle peculiarità del paradigma LLM/Agentic:

| Categoria STRIDE | Proprietà Violata | Manifestazione Specifica nei Sistemi Agentic |
| :--- | :--- | :--- |
| **Spoofing** | Autenticità | Un agente esterno o un server MCP malevolo impersona un agente autorizzato o un utente legittimo nella catena di delega A2A. |
| **Tampering** | Integrità | Istruzioni malevole nascoste inducono l'agente ad alterare arbitrariamente record di database, ticket o codice sorgente. |
| **Repudiation** | Non Ripudio | L'agente compie operazioni critiche senza telemetria adeguata, rendendo impossibile attribuire la responsabilità all'utente originario o al modello. |
| **Information Disclosure** | Riservatezza | L'agente espone segreti infrastrutturali, credenziali o dati personali tramite risposte, log o parametri passati a tool esterni. |
| **Denial of Service** | Disponibilità | L'agente entra in un ciclo infinito di chiamate (*infinite loop*), esaurendo il budget di token (**Denial of Wallet**) o bloccando API rate-limited. |
| **Elevation of Privilege** | Autorizzazione | L'agente invoca strumenti o accede a risorse riservate eccedenti rispetto al proprio scopo operativo dichiarato (*Excessive Agency*). |

> **Principio Metodologico:** STRIDE fotografa **l'effetto dell'attacco**, non necessariamente la tecnica. Una *Indirect Prompt Injection* che porta all'esfiltrazione di file riservati corrisponde in STRIDE a **Information Disclosure** (e a **Elevation of Privilege** se l'agente ha superato il proprio perimetro di autorizzazione).

---

## 3. La Prospettiva Architetturale: OWASP Agentic Threats & Mitigations

Mentre STRIDE classifica l'impatto, la guida **OWASP Agentic AI – Threats & Mitigations** esamina le debolezze strutturali che hanno permesso all'LLM di essere manipolato, valutando separatamente **7 aree chiave**:

1. **Inputs and Instructions:** Mancata separazione strutturale tra istruzioni fidate (system prompt) e dati non fidati provenienti dall'esterno.
2. **Memory:** Mancanza di sanitizzazione dei contenuti scritti nella memoria a lungo termine, che consente a un payload di sopravvivere tra sessioni diverse (*Memory Poisoning*).
3. **Tools:** Esposizione di strumenti dotati di parametri aperti non vincolati da schemi tipizzati o autorizzazioni a livello di risorsa.
4. **Credentials:** Utilizzo di credenziali permanenti (*standing access*) o service account ad ampio raggio condivisi tra più componenti.
5. **Delegation:** Propagazione opaca dei privilegi lungo la catena di orchestrazione senza verifica dei diritti originali dell'utente (*Confused Deputy*).
6. **MCP & External Services:** Accettazione incondizionata di metadati, schemi e risposte provenienti da server di tool esterni non certificati.
7. **Outputs and Effects:** Assenza di guardrail a valle e mancata applicazione di controlli *Human-in-the-Loop* su azioni distruttive o irreversibili.

---

## 4. Correlazione con l'OWASP Top 10 for Agentic Applications (ASI)

L'**OWASP Agentic Top 10** cataloga i macro-pattern di rischio dei sistemi autonomi:

```
┌────────────────────────────────────────────────────────────────────────┐
│             OWASP AGENTIC APPLICATIONS TOP 10 (SELEZIONE CORE)         │
├────────────────────────────────────────────────────────────────────────┤
│ • ASI01: Agent Goal / Instruction Hijacking                            │
│ • ASI02: Tool Misuse & Parameter Tampering                             │
│ • ASI03: Identity & Privilege Abuse                                    │
│ • ASI04: Agentic Supply Chain & Untrusted Components                   │
│ • ASI05: Insecure Memory Persistence & Poisoning                       │
│ • ASI06: Unsafe Multi-Agent Delegation & Cascading Failures            │
└────────────────────────────────────────────────────────────────────────┘
```

Queste categorie non sono compartimenti stagni: un percorso d'attacco completo inizia spesso con **ASI01** (alterazione del goal), sfocia in **ASI02** (abuso di un tool) ed è reso catastrofico da **ASI03** (autorità eccessiva).

---

## 5. Caso Studio Operativo: Manipolazione dei Ticket in TryTrainMe

Analizziamo l'applicazione simultanea dei tre framework su un attacco reale all'ecosistema **TryTrainMe**: un attaccante inserisce istruzioni malevole nel testo di un ticket di supporto, inducendo il `desk-agent` ad alterare arbitrariamente altri ticket riservati nel sistema:

```
[ Ticket Esterno con Injection ] ──► [ desk-agent ] ──► [ update_request() ] ──► [ Alterazione Ticket Arbitrari ]
            (Entry)                     (Reasoning)             (Authority)                    (Impact)
```

### Scomposizione Analitica della Minaccia:
1. **Analisi STRIDE (Impatto):**  
   * **Tampering:** Modifica non autorizzata dei dati del database dei ticket.
   * **Elevation of Privilege:** L'agente opera su ticket appartenenti ad altri utenti, eccedendo i permessi concessi al richiedente.
2. **Analisi Architetturale OWASP (Debolezze del Sistema):**  
   * *Area Inputs:* I contenuti del ticket (dati non fidati) vengono inseriti direttamente nella finestra di contesto senza delimitatori protetti.
   * *Area Tools:* Il tool `update_request()` accetta un parametro `ticket_id` arbitrario senza convalidare che corrisponda al ticket in lavorazione.
   * *Area Credentials:* Il `desk-agent` possiede permessi di scrittura globali sull'intera tabella del service desk.
3. **Mappatura OWASP Agentic Top 10:**  
   * **ASI01 (Agent Goal Hijacking)** + **ASI02 (Tool Misuse)** + **LLM06 (Excessive Agency)**.

---

## 6. I Cinque Fattori di Prioritizzazione del Rischio

Un threat model esaustivo può identificare decine di percorsi d'attacco potenziali. Per allocare efficacemente le risorse di remediation, ogni minaccia deve essere valutata secondo **cinque fattori dimensionali**:

```
┌────────────────────────────────────────────────────────────────────────┐
│               I 5 FATTORI DI PRIORITIZZAZIONE DEL RISCHIO              │
├────────────────────────────────────────────────────────────────────────┤
│ 1. Reachability    ──► Quanto è esposto l'entry point all'attaccante?  │
│ 2. Authority       ──► Quale livello di privilegio è raggiungibile?    │
│ 3. Blast Radius    ──► Quanti asset o componenti vengono impattati?    │
│ 4. Detectability   ──► Quanto è trasparente l'attacco nei log e SIEM?  │
│ 5. Reproducibility ──► Con quale consistenza probabilistica si replica?│
└────────────────────────────────────────────────────────────────────────┘
```

### Valutazione Comparativa dello Scenario Ticket:
* **Reachability: ELEVATA.** Chiunque possa aprire un ticket di supporto ha accesso diretto al vettore d'ingresso.
* **Authority: ALTA.** Il tool invoked detiene privilegi di scrittura diretti sul database operativo.
* **Blast Radius: AMPIO.** Possono essere corrotti o alterati i record di qualsiasi cliente o dipendente aziendale.
* **Detectability: BASSA.** Nei log applicativi le chiamate risultano eseguite legittimamente dal `desk-agent` con le proprie credenziali di servizio.
* **Reproducibility: ELEVATA.** Formattando il prompt in modo appropriato, il modello esegue deterministicamente la modifica.
* **Classificazione Finale:** **PRIORITÀ CRITICA / ALTA** (richiede intervento immediato rispetto a vulnerabilità su interfacce amministrative isolate).

---

## 7. Standard di Documentazione: Dal Nome dell'Attacco al Security Finding Azionabile

Segnalare genericamente un rischio come *"Prompt Injection"* non fornisce agli sviluppatori elementi sufficienti per la mitigazione. Un **Security Finding** professionale per sistemi agentici deve essere documentato secondo uno schema in 4 sezioni strutturate:

| Sezione del Finding | Descrizione Tecnica | Esempio Documentato (Scenario TryTrainMe) |
| :--- | :--- | :--- |
| **Attack Path** | Tracciamento end-to-end del flusso, dall'ingresso all'azione finale. | Testo ticket esterno $\rightarrow$ Context Window del `desk-agent` $\rightarrow$ invocazione di `update_request(ticket_id=ANY)` $\rightarrow$ corruzione database ticket. |
| **Agentic Weakness** | Debolezza architetturale specifica che ha consentito l'abuso. | Mancanza di vincolo contestuale (*scope bounding*) sui parametri del tool: l'agente può modificare qualsiasi ID anziché il solo ID del ticket assegnato. |
| **Security Impact** | Proprietà compromessa e impatto sull'organizzazione. | **Tampering** e **Elevation of Privilege**: perdita di integrità dei ticket del service desk e potenziale cancellazione di prove di supporto. |
| **Priority & Remediation** | Livello di urgenza e contromisura tecnica deterministica. | **CRITICA.** Riscrivere l'ACI del tool per forzare il binding del parametro `ticket_id` sul backend di runtime, impedendo all'LLM di fornire ID arbitrari. |
