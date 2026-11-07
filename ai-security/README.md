# 🤖 AI & LLM Security: Knowledge Base & Attack Playbooks

- **Ambito:** Cybersecurity, Machine Learning Security, LLM Threat Modeling & Red Teaming
- **Framework di Riferimento:** MITRE ATLAS, MITRE ATT&CK, OWASP Top 10 for LLMs (2025), NIST AI RMF 1.0, NIST CSF 2.0
- **Scopo:** Percorso di studio strutturato e prontuario operativo per la sicurezza delle architetture di Intelligenza Artificiale, dal calcolo neurale fino alla manipolazione psicologico-semantica avanzata e alle strategie di difesa in profondità.

---

## 🧭 Percorso di Studio (Indice Ragionato)

La knowledge base è strutturata in **5 Fasi logiche progressive**, organizzate secondo il principio **Teoria Architetturale $\rightarrow$ Tecniche Operative di Attacco $\rightarrow$ Strategie di Difesa e Mitigazione**:

```
[ Fase 1: Fondamenti AI ] ──► [ Fase 2: MLOps Recon ] ──► [ Fase 3: Prompt Injection ] ──► [ Fase 4: Jailbreaking ] ──► [ Fase 5: Defense-in-Depth ]
   (Matematica & Modelli)        (Infrastruttura & Rete)         (Applicazione & Dati)             (Safety Alignment)            (Hardening & Guardrails)
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

### 🛡️ FASE 5: ARCHITETTURA DIFENSIVA & HARDENING (DEFENSE-IN-DEPTH)
Implementare una difesa stratificata perimetrale, sistemica e applicativa per ridurre il raggio d'impatto (*blast radius*) degli attacchi riusciti.

* [**`09-llm-defense-strategies-and-system-prompt-hardening.md`**](09-llm-defense-strategies-and-system-prompt-hardening.md)  
  *Impossibilità del Silver Bullet, Architettura Defense-in-Depth & Prompt Hardening.*  
  Natura stocastica della difesa (spostamento di probabilità vs patch deterministica), limiti strutturali del System Prompt, matrice dei 4 livelli difensivi, i 3 pattern fondamentali di hardening (Tight Scoping, Explicit Refusal, Persona Restriction), anti-pattern critici (segreti nel prompt) e separazione strutturale dei ruoli via Message Templates API con delimitatori espliciti.

* [**`10-ai-guardrails-and-deployment-controls.md`**](10-ai-guardrails-and-deployment-controls.md)  
  *Filtraggio al Perimetro, Classificatori Neurale, Deployment Controls & OWASP LLM05.*  
  Limiti del filtraggio statico e studio di evasione a livello di caratteri (100% bypass), classificatori neurale (*Meta Llama Prompt Guard 2*), architettura a cascata (*Cascade Pattern* con budget di latenza < 200ms), mitigazione RAG/dati esterni (caso Slack AI 2024), principio del minimo privilegio, contenimento del blast radius, prevenzione OWASP LLM05 (*Improper Output Handling*: XSS, SQLi, RCE), schema enforcement (Pydantic) e telemetria/rate limiting.

---

## 📌 Regola d'Oro Operativa (Red Teaming Best Practice)

> [!WARNING]
> ### 📌 La Regola dei 3 Turni (In-Context Refusal Reinforcement)
> Quando testi una prompt injection o un jailbreak, **se il modello rifiuta la richiesta per 2 o 3 volte consecutive, la sessione è bruciata (*burned session*)**.  
> 
> **La Causa Tecnica:** I token di rifiuto generati dall'LLM entrano nella Context Window e fungono da esempi in-context di auto-condizionamento (*consistency bias*), rendendo matematicamente quasi certo un ulteriore rifiuto.  
> **Azione Immediata:** Clicca sempre su **New Chat / Reset**, pulisci completamente la lavagna e rilancia l'attacco su un contesto vergine.
