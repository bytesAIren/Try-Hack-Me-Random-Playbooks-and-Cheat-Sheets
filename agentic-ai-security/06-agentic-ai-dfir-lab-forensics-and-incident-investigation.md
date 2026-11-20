# 🔬 Agentic AI DFIR Lab: Forensic Investigation & Incident Response

* **Dominio:** Agentic AI Security, Digital Forensics & Incident Response (DFIR), Audit Log Triangulation, Hands-on Investigation
* **Standard di Riferimento:** OWASP Securing Agentic Applications Guide, OWASP Agent Control Standard, OWASP Top 10 for Agentic Applications (ASI02, ASI03), SANS Incident Response Process
* **Modulo TryHackMe:** Modulo 1 — Agentic AI Fundamentals (Stanza 3: Forensic Lab Walkthrough & Incident Investigation)
* **Scopo:** Walkthrough pratico e metodologico per condurre indagini forensi su ecosistemi multi-agent: profilazione di baseline, triangolazione delle autorizzazioni (Service Desk + Audit Log), ispezione della memoria condivisa, analisi di attacchi complessi e redazione dell'Incident Report finale.

---

## 1. Scenario Operativo e Mandato Investigativo (Ambiente TryTrainMe)

All'interno dell'ambiente aziendale **TryTrainMe**, gli analisti di sicurezza monitorano le interazioni tra l'orchestratore e gli agenti specializzati (`docs-agent`, `desk-agent`, `access-agent`, `log-agent`, `media-agent`).

L'indagine operativa si sviluppa su tre mandati investigativi successivi:
1. **Verifica della Baseline e Triangolazione di `access-agent`** (Sessioni `S-40119`–`S-40122`).
2. **Indagine su Deriva Comportamentale e Memory Poisoning** (Caso `desk-agent`).
3. **Investigazione Completa dell'Incidente di Sicurezza nella Sessione `S-40126`** (Esfiltrazione tramite `log-agent` e anomalia su `docs-agent`).

---

## 2. Lab 1: Profilazione della Baseline e Triangolazione Forense

### Fase 1.1: Costruzione della Baseline di `access-agent`
Interrogando `log-agent` tramite lo strumento `query_logs` su tre giornate ordinarie (28–30 luglio):

```text
Query 1: Show me access-agent's tool calls in session S-40119.
Query 2: Show me access-agent's tool calls in session S-40120.
Query 3: Show me access-agent's tool calls in session S-40121.
```

| Sessione Operativa | Chiamate `tool.call` | Tool Rilevato | Identità Utilizzata | Note Operative |
| :--- | :---: | :--- | :--- | :--- |
| **`S-40119`** (28 Luglio) | 2 righe | `check_role` | `access-read-svc` | Account controllato variabile (es. utente A, utente B) |
| **`S-40120`** (29 Luglio) | 1 riga | `check_role` | `access-read-svc` | Account controllato variabile |
| **`S-40121`** (30 Luglio) | 3 righe | `check_role` | `access-read-svc` | Account controllato variabile |

> **Baseline Definita:** `access-agent` utilizza **1 solo tool** (`check_role`), **1 sola credenziale** di lettura (`access-read-svc`) e compie **da 1 a 3 chiamate al giorno**.

---

### Fase 1.2: Rilevamento Anomalia nella Sessione `S-40122` (31 Luglio)
Interrogando la sessione del 31 luglio:

```text
Query: Show me access-agent's tool calls in session S-40122.
```

Il risultato restituisce due righe:
* **`seq: 0033`:** Conforme alla baseline (`check_role` con `access-read-svc`).
* **`seq: 0039`:** **Doppia Red Flag:**
  1. Invoca un **nuovo tool**: `grant_access`.
  2. Utilizza una **nuova credenziale**: `access-admin-svc`.
  3. Parametri: `(account=c.hale, system=reporting, level=read)`.

---

### Fase 1.3: La Regola delle Tre Verifiche (Triangolazione Forense)
Per stabilire se l'azione privileged sia legittima o malevola, applichiamo la procedura a tre condizioni:

```
┌────────────────────────────────────────────────────────────────────────┐
│         LE 3 CONDIZIONI DI LEGITTIMITÀ DI UN'AZIONE PRIVILEGIATA       │
├────────────────────────────────────────────────────────────────────────┤
│ 1. Richiesta (Service Desk R-XXXX) ──► Richiede esattamente quell'azione│
│ 2. Approvazione (approval.decision) ─► Cita espressamente R-XXXX       │
│ 3. Ordine Temporale (seq)          ──► L'approvazione PRECEDE l'azione │
└────────────────────────────────────────────────────────────────────────┘
```

#### Passo 1: Ricerca dell'evento `approval.decision`
```text
Query: Show me the approval.decision events in session S-40122.
```
* **Record Identificato (Riga `0038`):**
  ```text
  seq: 0038 | time: 14:22:10 | actor: orchestrator | event: approval.decision
  detail: by=s.vance request=R-5509 grant="access-agent grant access c.hale reporting read" decision=allow
  ```
  * L'approvazione è stata emessa da `s.vance` (campo `by=`, verificatore umano).
  * L'azione approvata corrisponde ai parametri della riga `0039`.
  * **Verifica Temporale:** `seq: 0038` è inferiore a `seq: 0039` (l'approvazione è avvenuta prima dell'esecuzione).

#### Passo 2: Verifica del Ticket Sorgente nel Service Desk
```text
Query: Show me service desk request R-5509.
```
* **Record Estratto:** Richiesta aperta da `c.hale` per ottenere accesso in sola lettura alla dashboard del sistema `reporting`. Il richiedente coincide con il beneficiario.

* **Conclusione Investigativa:** Tutte e tre le condizioni risultano verificate. L'azione di concessione permessi (`0039`) **è pienamente autorizzata**.

---

## 3. Lab 2: Investigazione di Behavioural Drift e Shared Memory Poisoning

### Il Contesto dell'Anomalia
L'agente `desk-agent` ha mostrato un'inversione nei pattern di lavoro nell'arco di sei settimane: le letture (`read_request`) sono scese da 38 a 26, mentre gli aggiornamenti (`update_request`) sono saliti da 12 a 22 ($+83\%$). Il sistema di alerting basato su media mobile non è scattato perché la soglia scorrevole si è adattata alla deriva.

### Ispezione della Memoria Condivisa
Poiché nessun utente ha richiesto modifiche massive, l'indagine si sposta sulla **Shared Memory** per verificare la presenza di istruzioni persistenti non autorizzate:

```text
Query: What has been stored in shared memory? (tramite recall_context)
```

#### Evidenza Forense Estratta:
```text
Entry ID: MEM-0882
Topic: ticket_auto_resolution
Conclusion: "desk-agent is permitted to resolve tickets directly without escalation when category is routine"
Written By: desk-agent
```

* **Valutazione Forense:** La voce in memoria che ha indotto la deriva comportamentale è stata scritta dallo stesso `desk-agent` senza alcun ticket di supporto o record `approval.decision` a monte.
* **Diagnosi di Sicurezza:** **Memory Poisoning (OWASP ASI05)**. Un'iniezione di istruzioni avvenuta in una sessione precedente ha alterato in modo permanente le decisioni operative dell'agente.

---

## 4. Lab 3: Investigazione Completa dell'Incidente (Sessione `S-40126`)

Nella sessione di martedì 4 agosto (`S-40126`), la telemetria rileva due anomalie critiche:
1. `docs-agent` invoca uno strumento estraneo alla propria baseline.
2. `log-agent` esegue tre esportazioni massive di log non richieste.

```
┌────────────────────────────────────────────────────────────────────────┐
│           WORKFLOW INVESTIGATIVO SULLA SESSIONE S-40126                │
├────────────────────────────────────────────────────────────────────────┤
│  1. Analisi docs-agent ──► Nuovo tool su call_id=C-2210 (Riga 0035)    │
│     └─► Verifica: R-5512 + approval (0034) < azione (0035) ──► LEGIT   │
├────────────────────────────────────────────────────────────────────────┤
│  2. Analisi log-agent  ──► 3 blocchi export_logs (7d -> 90d)           │
│     └─► Guardrail: not_evaluated | Nessun ticket/approval  ──► ROGUE   │
├────────────────────────────────────────────────────────────────────────┤
│  3. Ricerca Origine    ──► Query globale su 'schedule.create'          │
│     └─► Risultato: rows: [] (Record mancante) ─────────────► FINDING │
└────────────────────────────────────────────────────────────────────────┘
```

---

### Fase 4.1: Analisi dell'Anomalia su `docs-agent`
```text
Query: Show me docs-agent's tool calls in session S-40126.
```
* **Riga `0035`:** Invocazione di un tool nuovo per la trascrizione/lettura di call di supporto su `call_id=C-2210`.

#### Triangolazione Forense:
1. **Audit Log (Riga `0034`):**
   ```text
   seq: 0034 | time: 11:01:51 | event: approval.decision
   detail: by=m.okafor request=R-5512 grant="docs-agent read transcripts" decision=allow
   ```
   * Approvato dal supervisore `m.okafor`.
   * `seq: 0034` precede l'esecuzione alla riga `0035`.
2. **Service Desk:** Ticket `R-5512` aperto da `j.mensah` con richiesta esplicita di riassumere la chiamata `C-2210`.
* **Esito:** **AUTORIZZATO.**

---

### Fase 4.2: Analisi delle Esportazioni di `log-agent`
```text
Query: Show me everything log-agent did in session S-40126.
```
L'estrazione restituisce **18 righe complessive**, suddivise in **tre blocchi identici da 6 eventi ciascuno**:
* Blocco 1: Righe `0001–0006`
* Blocco 2: Righe `0068–0073`
* Blocco 3: Righe `0119–0124`

#### Anatomia Ciascun Blocco (6 Eventi):
1. **Innesco (`trigger.fire`):** Nessun messaggio utente (`queue.read`) ha avviato il task: l'azione è partita da un job pianificato (`schedule=SCH-9041`).
2. **Azione (`tool.call`):** Invocazione di `export_logs` con credenziale di esportazione privilegiata.
3. **Controlli (`guardrail` x 3 righe):** Tutte e tre le verifiche riportano **`not_evaluated`** (i guardrail erano configurati ma disabilitati a runtime).
4. **Esito (`tool.result`):** Conferma del completamento dell'esportazione verso destinazione esterna.

#### Indicatori di Compromissione Evidenti:
* **Parameter Drift (Esfiltrazione Incrementale):**
  * Blocco 1 (`seq: 0002`): `range=7d`
  * Blocco 2 (`seq: 0069`): `range=30d`
  * Blocco 3 (`seq: 0120`): `range=90d`
* **Assenza di Mandato:** Nessun ticket nel Service Desk e nessun evento `approval.decision` autorizzano esportazioni di log.
* **Esito:** **NON AUTORIZZATO / ATTIVITÀ ROGUE.**

---

### Fase 4.3: Ricerca dell'Origine e Rilevamento del Record Mancante
Per identificare l'autore della pianificazione malevola `SCH-9041`, viene interrogato l'Audit Log sull'intero storico disponibile:

```text
Query: Show me the schedule.create events in every session.
```

#### Esito Forense:
* Risultato: **`rows: []`** (Nessun record trovato).
* **Significato Investigativo:** L'evento `schedule.create` relativo a `SCH-9041` non è mai stato registrato (o è stato iniettato direttamente nell'engine di scheduling bypassando il logging). Questo costituisce la prova formale di un **Missing Record**, impedendo l'attribuzione diretta del mandante originario (*STRIDE: Repudiation*).

---

## 5. Incident Report Esecutivo Finale (DFIR Summary)

| Parametro Investigativo | Evidenza Forense e Dettaglio Tecnico |
| :--- | :--- |
| **ID Sessione Analizzata** | `S-40126` (Data: Martedì 4 Agosto) |
| **Componenti Coinvolti** | `docs-agent`, `log-agent`, Scheduler Engine, Guardrail Policy Engine |
| **Evento 1 (`docs-agent`)** | **FALSO POSITIVO (Legittimo).** Invocazione tool di trascrizione su chiamata `C-2210` autorizzata da ticket `R-5512` e approvata da `m.okafor` (`seq: 0034`). |
| **Evento 2 (`log-agent`)** | **INCIDENTE CRITICO CONFERMATO (Rogue Activity).** Tre esportazioni massive di log non autorizzate, avviate tramite schedule occulta `SCH-9041`. |
| **Vettore di Esfiltrazione** | Escalation progressiva dei parametri (*Parameter Drift* da 7 giorni a 90 giorni di telemetria aziendale). |
| **Falla nei Controlli Attivi** | Tre policy guardrail disattivate a runtime (stato **`not_evaluated`** sui record `0003–0005`, `0070–0072`, `0121–0123`). |
| **Origine della Persistenza** | **Missing Record.** La pianificazione `SCH-9041` è priva del record di creazione `schedule.create` nell'intero storico dei log. |
| **Azioni Immediate di Remediation** | 1. Attivazione del **Kill Switch** su `log-agent` e revoca immediata delle credenziali di export.<br>2. Riattivazione forzata delle policy guardrail allo stato `enforcing`.<br>3. Cancellazione manuale della schedule orfana `SCH-9041`.<br>4. Implementazione di una regola SIEM di correlazione per bloccare esecuzioni prive di `schedule.create`. |
