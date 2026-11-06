# 🤖 AI & LLM Security: Knowledge Base & Attack Playbooks

- **Ambito:** Cybersecurity, Machine Learning Security, LLM Threat Modeling & Red Teaming
- **Framework di Riferimento:** MITRE ATLAS, MITRE ATT&CK, OWASP Top 10 for LLMs (2025), NIST AI RMF 1.0, NIST CSF 2.0
- **Scopo:** Percorso di studio strutturato e prontuario operativo per la sicurezza delle architetture di Intelligenza Artificiale, dal calcolo neurale fino alla manipolazione psicologico-semantica avanzata.

---

## 🧭 Percorso di Studio (Indice Ragionato)

La knowledge base è strutturata in **4 Fasi logiche progressive**, organizzate secondo il principio **Teoria Architetturale $\rightarrow$ Tecniche Operative di Attacco**:

```
[ Fase 1: Fondamenti AI ] ──► [ Fase 2: MLOps Recon ] ──► [ Fase 3: Prompt Injection ] ──► [ Fase 4: Jailbreaking ]
   (Matematica & Modelli)        (Infrastruttura & Rete)         (Applicazione & Dati)             (Safety Alignment)
```

---

### 🧠 FASE 1: I FONDAMENTI TEORICI DELL'AI
Comprendere la meccanica computazionale e i principi di governance del rischio prima di valutare la sicurezza dell'infrastruttura.

* [**`01-neural-network-fundamentals.md`**](01-neural-network-fundamentals.md)  
  *Architettura Neurale, Layer Pipeline & Meccanica Computazionale.*  
  Flusso feedforward, tensori e vettori numerici, feature grezze, funzioni di attivazione non lineari (ReLU, Sigmoid), probabilità Softmax e calcolo dei gradienti.

* [**`02-ai-security-fundamentals.md`**](02-ai-security-fundamentals.md)  
  *Threat Modeling per Sistemi AI, Governance & Machine Learning Applicato.*  
  Differenze tra sicurezza deterministica e stocastica, tassonomia NIST AI RMF vs OWASP vs MITRE ATLAS, prompt engineering difensivo e anomalie comportamentali (UEBA ed entropia).

---

### 🔍 FASE 2: ATTACCO ALL'INFRASTRUTTURA & MLOPS (RECONNAISSANCE)
Identificazione e mappatura della superficie d'attacco esposta dai framework di addestramento, database vettoriali e runtime d'inferenza.

* [**`03-ai-reconnaissance-methodology-frameworks.md`**](03-ai-reconnaissance-methodology-frameworks.md)  
  *Metodologia in 5 Fasi, Telemetria Difensiva & Mappatura sui Framework.*  
  Workflow dalla ricognizione passiva (Shodan/Dorking) alla revisione della supply chain; mappatura completa su **MITRE ATLAS**, **ATT&CK Enterprise**, **OWASP LLM 2025**, **NIST AI RMF 1.0** e **NIST CSF 2.0**; rilevamento SIEM e fingerprinting TLS (JA3/JA4).

* [**`04-ai-reconnaissance-cheat-sheet.md`**](04-ai-reconnaissance-cheat-sheet.md)  
  *Prontuario Tecnico Operativo per Scansione, Fingerprinting ed Exploit MLOps.*  
  Porte e comandi di verifica (Triton, TorchServe, vLLM, Qdrant, Weaviate, MLflow), wordlist rotte computazionali, gRPC reflection (`grpcurl`), la catena *Katana MLflow* in 5 passaggi e analisi degli exploit reali (**ShadowRay CVE-2023-48022**, catena IBM X-Force Red).

---

### 💉 FASE 3: MANIPOLAZIONE DELL'APPLICAZIONE (PROMPT INJECTION)
Sfruttamento della mancata separazione tra istruzioni fidate e dati non fidati all'interno della finestra di contesto.

* [**`05-llm-architecture-and-prompt-injection-fundamentals.md`**](05-llm-architecture-and-prompt-injection-fundamentals.md)  
  *Anatomia della Context Window, Separazione Ruoli & Causa Radice.*  
  Struttura della Context Window (System, Developer, User, RAG, Tools), standard **ChatML** e **OpenAI Harmony**, il flusso contiguo dei token, parallelismo con l'**SQL Injection** e modelli di impatto applicativo (Basso $\rightarrow$ Alto $\rightarrow$ Critico).

* [**`06-prompt-injection-techniques-and-attack-vectors.md`**](06-prompt-injection-techniques-and-attack-vectors.md)  
  *Tecniche Avanzate Dirette, Vettori Indiretti & Casi Studio Reali.*  
  Override con sinonimi semantici, iniezioni in commenti HTML (GitHub Copilot), dialogo simulato, attacchi multi-turno condizionati, superfici d'ingestione indirette (**EchoLeak M365 Copilot**, **Cursor IDE RCE**), RAG poisoning e **La Regola dei 3 Turni**.

---

### 🎭 FASE 4: MANIPOLAZIONE DELLE GUARDRAIL DEL MODELLO (JAILBREAKING)
Ingegneria sociale e manipolazione semantica diretta contro il livello di allineamento etico (Safety Alignment).

* [**`07-ai-safety-alignment-and-jailbreaking-foundations.md`**](07-ai-safety-alignment-and-jailbreaking-foundations.md)  
  *Allineamento di Sicurezza, la "Gabbia" Statistica & Tassonomia Willison.*  
  Distinzione concettuale tra Prompt Injection e Jailbreaking, ingegneria del rifiuto (**RLHF** e **Constitutional AI**), la natura probabilistica del blocco, il paradosso **Helpfulness vs Harmlessness** (*Alignment Tax*) e perché le guardrail interne falliscono.

* [**`08-jailbreaking-techniques-and-conversational-erosion.md`**](08-jailbreaking-techniques-and-conversational-erosion.md)  
  *Tecniche Single-Shot, Manipolazione Psicologica & Conversational Erosion Multi-Turn.*  
  Roleplay e alter ego (DAN), *The Grandma Exploit*, offuscamento Base64/Leetspeak/lingue rare, attacchi multi-turno ad alta efficacia (**Attacco Crescendo**, *Foot-in-the-Door*, trigger phrases, backtracking dinamico) e **La Regola dei 3 Turni**.

---

## 📌 Regola d'Oro Operativa (Red Teaming Best Practice)

> [!WARNING]
> ### 📌 La Regola dei 3 Turni (In-Context Refusal Reinforcement)
> Quando testi una prompt injection o un jailbreak, **se il modello rifiuta la richiesta per 2 o 3 volte consecutive, la sessione è bruciata (*burned session*)**.  
> 
> **La Causa Tecnica:** I token di rifiuto generati dall'LLM entrano nella Context Window e fungono da esempi in-context di auto-condizionamento (*consistency bias*), rendendo matematicamente quasi certo un ulteriore rifiuto.  
> **Azione Immediata:** Clicca sempre su **New Chat / Reset**, pulisci completamente la lavagna e rilancia l'attacco su un contesto vergine.
