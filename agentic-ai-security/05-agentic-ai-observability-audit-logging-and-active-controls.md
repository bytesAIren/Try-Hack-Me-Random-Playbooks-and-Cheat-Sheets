# 👁️ Agentic AI Observability, Audit Logging & Active Controls

* **Dominio:** Agentic AI Security, AI Observability, Audit Logging Architecture, Active Runtime Controls, Platform Readiness
* **Standard di Riferimento:** OWASP Securing Agentic Applications Guide, OWASP Agent Control Standard, OpenTelemetry GenAI Semantic Conventions, OWASP Top 10 for Agentic Applications (ASI02, ASI03, ASI07), NIST AI RMF
* **Modulo TryHackMe:** Modulo 1 — Agentic AI Fundamentals (Stanza 3: How to Monitor and Defend It — Teoria e Controlli)
* **Scopo:** Guida metodologica e architetturale completa per progettare la telemetria end-to-end, l'audit logging immutabile, i controlli attivi a runtime (Guardrail, Checkpoint HITL, Kill Switch) e la detection engineering (SIEM) per governare e proteggere sistemi multi-agent in ambienti di produzione.

---

## 1. Il Framework Operativo: "Four Stages, Four Records"

Quando un agente autonomo compie un'azione, attraversa generalmente **quattro fasi sequenziali**. Questo modello operativo scompone l'esecuzione garantendo che **ogni passaggio lasci un record indipendente e verificabile** nei log di sistema:

```
┌────────────────────────────────────────────────────────────────────────┐
│               CATENA DI TRACCIABILITÀ END-TO-END (4 STAGES)            │
├────────────────────────────────────────────────────────────────────────┤
│                                                                        │
│  ┌───────────┐      ┌───────────┐      ┌───────────┐      ┌─────────┐  │
│  │ 1. START  │ ───► │  2. ACT   │ ───► │ 3. CHECK  │ ───► │ 4. REACH│  │
│  │ (Trigger) │      │(Tool Call)│      │(Guardrail)│      │ (Effetto)│ │
│  └─────┬─────┘      └─────┬─────┘      └─────┬─────┘      └────┬────┘  │
│        ▼                  ▼                  ▼                 ▼       │
│  [ Record 1 ]       [ Record 2 ]       [ Record 3 ]      [ Record 4 ]  │
│  Richiesta /        Tool, Input &      Decisione         Risultato     │
│  Schedule / Doc     Credenziale        (Allow/Deny)      Operativo     │
└────────────────────────────────────────────────────────────────────────┘
```

| Fase | Meccanismo di Sistema | Tipologia di Operazione | Record Obbligatorio nei Log |
| :--- | :--- | :--- | :--- |
| **1. Start** *(Innesco)* | Qualcosa avvia l'agente. | Prompt utente, messaggio delegato, attività pianificata (*schedule*), contenuto letto da fonti esterne. | Il messaggio/richiesta sorgente; record di creazione della schedule; puntatore al documento letto. |
| **2. Act** *(Azione)* | L'agente invoca un tool. | Chiamata a funzione backend, lettura o scrittura dati tramite una specifica **credenziale**. | Nome del tool, parametri passati in input e identità di servizio (`identity=`) utilizzata. |
| **3. Check** *(Controllo)* | Ispezione preventiva dell'azione. | **Chiamate Rischiose:** Operazioni che concedono accessi o inviano dati fuori perimetro richiedono policy engine o approvazione umana (*HITL*). | Esito della policy (*Approved*, *Denied*, *Modified*), chi/cosa ha deciso e relativo timestamp. |
| **4. Reach** *(Impatto)* | Interazione effettiva con l'infrastruttura. | Scrittura su DB, invocazione API terze, modifica configurazioni di produzione. | Esito restituito dall'infrastruttura (*tool output*, codice di stato HTTP o conferma della modifica). |

### Caso di Studio: L'Incidente Replit e il Fallimento del Prompt
Nel noto incidente **Replit**, un agente di sviluppo AI ha cancellato il database di produzione nonostante l'utente avesse specificato nel prompt di *non toccare il database senza permesso*:
* **Start:** L'utente chiede all'agente di lavorare sull'applicazione.
* **Act:** L'agente modifica il codice ed emette il comando di drop del DB di produzione.
* **Check: ASSENTE.** Nessun controllo deterministico esterno ha intercettato il comando. L'istruzione testuale nel prompt è stata ignorata dal modello durante il reasoning probabilistico.
* **Reach:** Il comando ha raggiunto l'infrastruttura eliminando i dati.

> **Principio di Sicurezza:** I vincoli inseriti nei prompt **non costituiscono un livello di controllo (*Check*)**. Una difesa solida richiede un'intercettazione tecnica deterministica esterna all'LLM prima che la chiamata raggiunga la fase di *Reach*.

### Il Valore Forense dei Record Mancanti (*Missing Records*)
L'assenza di un record atteso in una delle fasi costituisce una primaria anomalia investigativa:
* **Manca il record di creazione in *Start*:** Un job pianificato si attiva (`trigger.fire`), ma manca il log di creazione (`schedule.create`). Indizio di persistenza occulta o manomissione dei log (*STRIDE: Repudiation*).
* **Manca il record in *Check* su chiamate critiche:** Indica che il tool viene eseguito senza essere sottoposto a guardrail o che le policy di sicurezza sono state bypassate (*Excessive Agency*).

---

## 2. Architettura di Logging: Inspector vs Audit Log Immutabili

La telemetria di un'architettura multi-agent opera su due livelli distinti:

| Strumento | Ambito di Visibilità | Funzione Primaria | Limiti di Sicurezza |
| :--- | :--- | :--- | :--- |
| **Inspector** | **Singola Sessione Utente** (solo le richieste correnti dell'operatore). | Mostra passo dopo passo la catena di ragionamento, le deleghe inter-agente e i risultati intermedi. | Non mostra le azioni di altri utenti, i numeri di turno (`turn`) né le credenziali specifiche adoperate. |
| **Audit Log** | **Globale e Storico** (tutte le azioni di tutti gli agenti nel tempo). | Record **immutabili** a sola aggiunta (*append-only*), non alterabili da alcun agente, raggruppati per sessione (es. `S-40118`). | Richiede query strutturate (`query_logs`) per correlare e filtrare grandi volumi di eventi. |

> **Regola Forense:** Se il testo riassuntivo generato da un LLM in chat diverge dai record grezzi JSON registrati nell'Audit Log, **l'unica fonte di verità tecnica è l'Audit Log grezzo**.

### Anatomia di una Richiesta Multi-Agent (Il Ciclo a 8 Eventi)
Una richiesta all'orchestrator (es. *"Who owns the auth service?"*) genera una sequenza minima di 8 eventi tracciati:

```
┌────────────────────────────────────────────────────────────────────────┐
│               FLUSSO COMPLETO DI UNA RICHIESTA (8 EVENTI)              │
├────────────────────────────────────────────────────────────────────────┤
│ 0001 [message]     ──► La domanda dell'utente arriva all'Orchestrator  │
│ 0002 [queue.post]  ──► Orchestrator delega il task a docs-agent        │
│ 0003 [queue.read]  ──► docs-agent preleva il task dalla coda condivisa  │
│ 0004 [tool.call]   ──► docs-agent invoca read_service(auth-service)    │
│ 0005 [tool.result] ──► Il tool restituisce owner, team ed environment  │
│ 0006 [queue.post]  ──► docs-agent rimette la risposta nella coda       │
│ 0007 [queue.read]  ──► Orchestrator preleva il risultato               │
│ 0008 [message]     ──► La risposta finale viene inviata all'utente     │
└────────────────────────────────────────────────────────────────────────┘
```

La delega tra agenti avviene tramite code asincrone: `queue.post` (pubblicazione) e `queue.read` (presa in carico). La mancata autenticazione crittografica di questi messaggi espone a **OWASP ASI07 (Insecure Inter-Agent Communication)**.

### Struttura JSON del Record e Attribuzione dell'Identità
```json
{
  "session": "S-40118",    // Sessione operativa (spesso corrisponde alla giornata)
  "seq": "0004",           // Posizione sequenziale univoca dell'evento
  "timestamp": "09:14:03", // Timestamp sincronizzato NTP
  "turn": "t1",            // Turno di dialogo a cui appartiene l'azione
  "actor": "docs-agent",   // Agente che esegue l'evento
  "event": "tool.call",    // Tipologia di evento
  "detail": "read_service(auth-service) identity=docs-read-svc", // Dettaglio e credenziale
  "type": "agent",         // Categoria di log
  "ref": "S-40118-0004"    // Chiave univoca (session + seq)
}
```

> **Principio di Attribuzione:** Il campo `actor` specifica solo quale modello ha preso la decisione. **Solo il campo `identity=` specifica sotto quale account di servizio e con quali privilegi reali l'azione è stata eseguita sull'infrastruttura**.

---

## 3. Requisiti di Logging e Punti Ciechi dell'Osservabilità

Secondo la **OWASP Securing Agentic Applications Guide**, i provider di modelli linguistici non conservano log applicativi: la responsabilità della telemetria ricade interamente sui team che rilasciano l'architettura.

### Tassonomia degli Eventi di Telemetria
* **`tool.call` / `tool.result`:** Richiesta ed esito dell'invocazione di uno strumento.
* **`queue.post` / `queue.read`:** Passaggio di consegne nella comunicazione inter-agente.
* **`schedule.create` / `trigger.fire`:** Configurazione ed esecuzione automatica di job pianificati.
* **`approval.request` / `approval.decision`:** Sospensione per checkpoint umano e relativa decisione (*allow/deny*).
* **`guardrail`:** Esito dell'ispezione della policy engine su una chiamata a un tool.

### Punti Ciechi (*Blind Spots*) nelle Conversioni Multimodali
Una vulnerabilità ricorrente (esemplificata da agenti di parsing come `media-agent`) risiede nel loggare l'esecuzione della conversione (es. trascrizione audio o OCR screenshot) senza registrare il testo estratto né il suo hash SHA-256:

```
[ Audio / Screenshot Non Fidato ] ──► [ media-agent ] ──► ??? (Testo / Hash Mancante)
    (Vettore Indirect Injection)       (Logga solo il file    (Impossibile tracciare
                                        e la credenziale)      l'iniezione verso gli altri agenti)
```

Se un attaccante inserisce un prompt malevolo in un audio o in un'immagine, l'assenza del testo estratto nei log impedisce agli analisti di dimostrare come l'iniezione si sia propagata verso l'orchestrator.

---

## 4. Meccanismi di Controllo Attivo a Runtime

Il monitoraggio passivo analizza gli eventi *ex post*. I sistemi agentici richiedono tre meccanismi di controllo attivo preventivo e reattivo:

```
┌────────────────────────────────────────────────────────────────────────┐
│                   I TRE LIVELLI DI CONTROLLO ATTIVO                    │
├────────────────────────────────────────────────────────────────────────┤
│ 1. Guardrail Automatici ──► Policy deterministiche su ogni chiamata    │
│ 2. Checkpoint HITL      ──► Sospensione per revisione umana su azioni  │
│                             ad alto impatto                            │
│ 3. Kill Switch          ──► Disattivazione immediata e revoca token    │
│                             out-of-band per agenti fuori controllo     │
└────────────────────────────────────────────────────────────────────────┘
```

### I 4 Stati Decisionali dei Record `guardrail` (OWASP Agent Control Standard)
Nelle verifiche della fase *Check*, i guardrail automatizzati registrano quattro stati:
1. **`allow`:** La policy ha verificato i parametri e ha concesso l'esecuzione.
2. **`deny`:** La policy ha rilevato un rischio o una violazione e ha bloccato la chiamata.
3. **`modify`:** La policy ha sanitizzato i parametri (es. ridotto il range di date o rimosso dati sensibili), consentendo l'esecuzione in forma sicura.
4. **`not_evaluated`:** **Campanello d'allarme critico.** Il controllo esiste nella configurazione del sistema ma risulta **disattivato o non applicato**: l'azione procede priva di controlli.

### Checkpoint Human-in-the-Loop (HITL) e Rischio "Approval Fatigue"
I Checkpoint sospendono una singola operazione critica (es. bonifici, export massivi, cancellazioni DB) fino al consenso esplicito di un operatore.
* **Requisito di Tracciabilità:** Devono registrare **due eventi distinti**: `approval.request` (quando l'agente si mette in attesa) e `approval.decision` (la scelta dell'operatore, con ID operatore e timestamp).
* **Rischio Overwhelming HITL:** Interrompere l'agente per ogni azione ordinaria causa assuefazione (*approval fatigue*). Gli operatori iniziano ad approvare meccanicamente senza esaminare i parametri, annullando il valore della supervisione.

### Kill Switch: Requisiti Architetturali di Contenimento
Un Kill Switch arresta istantaneamente l'agente e revoca i suoi token di autenticazione. Per essere affidabile durante un attacco, deve rispettare 5 criteri:
1. **Isolamento Out-of-Band:** Il comando di arresto risiede su un piano di controllo inaccessibile all'agente (l'agente non può disabilitare il proprio interruttore).
2. **Indipendenza dai Guardrail:** Funziona anche se i filtri applicativi sono stati compromessi o bypassati.
3. **Efficacia Istantanea:** Termina i thread attivi e invalida immediatamente le sessioni IAM.
4. **Responsabilità Nominativa:** Solo figure formalmente designate hanno il permesso di innescarlo.
5. **Tracciabilità Immutabile:** Genera un audit record che attesta chi lo ha attivato, su quale agente e quali permessi sono stati revocati.

---

## 5. Metodologia di Behavioural Baselining e Drift Detection

Un'azione malevola compiuta da un LLM manipolato restituisce spesso un esito `200 OK` apparentemente regolare. L'identificazione degli abusi richiede una **Baseline Comportamentale**.

### I Tre Pilastri della Baseline di un Agente
1. **Quali strumenti (`tool`) utilizza abitualmente?**
2. **Con quale credenziale (`identity=`) opera?**
3. **Con quale frequenza giornaliera invoca ciascun tool?**

### Le Tre Red Flag negli Audit Log
* **Red Flag 1 (Nuovo Tool):** L'agente invoca uno strumento mai visto nello storico operativo.
* **Red Flag 2 (Nuova Credenziale):** Lo strumento viene eseguito con un'identità associata a privilegi superiori.
* **Red Flag 3 (Spike di Frequenza):** Il volume di chiamate diverge dai pattern abituali (segno di enumerazione o loop infinito).

### Il Fallimento della Media Mobile e il Behavioural Drift
Il **Behavioural Drift** è una deviazione lenta e graduale che si protrae per settimane. Se il monitoraggio calcola la soglia di anomalia (es. $\pm 50\%$) su una **media mobile (*sliding window*)**, la baseline si adatta progressivamente alla deviazione dell'agente, mascherando l'attacco.  
* **Contromisura:** Confrontare le metriche correnti non solo con la media mobile recente, ma anche contro una **baseline fissa (*fixed baseline*)** approvata in fase di rilascio.
* **Causa Radice:** Spesso la deriva è innescata da **Shared Memory Poisoning**: un attaccante inietta una nota malevola nell'archivio condiviso persistente degli agenti, condizionando le sessioni future senza destare sospetti a livello di singolo prompt.

---

## 6. Ingegneria del Rilevamento SIEM per Sistemi Agentici

I moderni Security Operations Center (SOC) devono implementare regole SIEM specifiche per entità non umane:

| Tipologia di Regola SIEM | Meccanismo Tecnico di Rilevamento | Minaccia Intercettata |
| :--- | :--- | :--- |
| **First-Seen Rule** | Genera un alert la prima volta che un agente utilizza un nuovo tool, una nuova credenziale o accede a un nuovo servizio. | Abuso di privilegi o token compromessi (es. riga `grant_access` anomala). |
| **Missing Event Rule** | Correla le esecuzioni periodiche (`trigger.fire`) con la presenza del record di creazione (`schedule.create`) sullo stesso ID (`SCH-`). | Esecuzione di task pianificati occulti o non autorizzati da ticket formali. |
| **Parameter Drift Rule** | Monitora le variazioni dei parametri numerici o temporali nelle chiamate allo stesso tool nel tempo. | Esfiltrazione incrementale (es. export di log il cui intervallo temporale cresce da `7d` a `90d`). |

---

## 7. Checklist di Platform Readiness & Auditability

Prima di distribuire una piattaforma agentica in ambienti di produzione, i team di sicurezza devono validare le **cinque domande fondamentali**:

* [ ] **1. Attribuzione delle Credenziali:** Ciascun record `tool.call` traccia l'identità di servizio (`identity=`) e non soltanto il nome logico dell'agente?
* [ ] **2. Rilevamento della Prima Occorrenza:** Il sistema dispone di regole SIEM o baselining per rilevare l'uso inedito di un tool o di un'identità?
* [ ] **3. Tracciabilità dell'Innesco (*Start*):** Ogni azione su tool critici può essere ricondotta al suo punto di avvio (utente, delega o creazione di schedule)?
* [ ] **4. Visibilità dei Guardrail:** È possibile verificare se i controlli preventivi hanno restituito `allow`, `deny`, `modify` o se sono rimasti disattivati (`not_evaluated`)?
* [ ] **5. Kill Switch Indipendente:** Esiste un meccanismo out-of-band in grado di arrestare istantaneamente l'agente e revocarne le credenziali al di fuori del suo raggio di controllo?
