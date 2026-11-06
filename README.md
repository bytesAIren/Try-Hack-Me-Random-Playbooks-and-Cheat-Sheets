# 🛡️ TryHackMe & Red Teaming: Playbooks, Cheat Sheets & Notes

Archivio personale di appunti, metodologie operative e cheat sheet tecnici sviluppati durante le stanze di **TryHackMe**, lab di Penetration Testing e studi specialistici di **AI Security**.

---

## 🗂️ Struttura del Repository

Il repository è organizzato in tre macro-aree specialistiche:

```text
.
├── 🤖 ai-security/          # Architetture neurali, ricognizione MLOps, Prompt Injection e Jailbreaking
├── 🌐 web-security/         # Vulnerabilità web applicative, standard OWASP ed exploit
└── 🧭 playbooks/            # Metodologie operative end-to-end (Recon, Foothold, PrivEsc)
```

---

## 📚 Moduli e Aree Tematiche

### 1. 🤖 [AI & LLM Security](ai-security/README.md)
Un percorso didattico e operativo completo suddiviso in 4 fasi logiche:
* **Fase 1 - Fondamenti:** [01. Neural Network Fundamentals](ai-security/01-neural-network-fundamentals.md) e [02. AI Security Fundamentals](ai-security/02-ai-security-fundamentals.md).
* **Fase 2 - MLOps Recon:** [03. Methodology & Frameworks](ai-security/03-ai-reconnaissance-methodology-frameworks.md) e [04. Technical Cheat Sheet](ai-security/04-ai-reconnaissance-cheat-sheet.md).
* **Fase 3 - Prompt Injection:** [05. Architecture & Vulnerabilities](ai-security/05-llm-architecture-and-prompt-injection-fundamentals.md) e [06. Advanced Techniques & Attack Vectors](ai-security/06-prompt-injection-techniques-and-attack-vectors.md).
* **Fase 4 - Jailbreaking:** [07. Safety Alignment & Foundations](ai-security/07-ai-safety-alignment-and-jailbreaking-foundations.md) e [08. Techniques & Conversational Erosion](ai-security/08-jailbreaking-techniques-and-conversational-erosion.md).

👉 *Consulta il [README di AI Security](ai-security/README.md) per l'indice dettagliato e le mappe concettuali.*

---

### 2. 🌐 [Web Application Security](web-security/)
Appunti e prontuari sulle vulnerabilità applicative web e standard OWASP Top 10:
* [**Application Design Flaws**](web-security/application-design-flaws.md) — Flaws di architettura, verbose errors, hidden API e misconfigurazioni.
* [**SSTI - Object Traversal & Introspection Cheat Sheet**](web-security/ssti-object-traversal.md) — Server-Side Template Injection, introspezione di oggetti e catene di exploit.

---

### 3. 🧭 [Operator's Playbooks](playbooks/)
Linee guida metodologiche end-to-end per attività di red teaming e lab:
* [**The Operator's Playbook**](playbooks/the-operators-playbook.md) — Dalla ricognizione di rete e scansione delle porte, al foothold iniziale, fino all'escalation dei privilegi (PrivEsc) e post-exploitation.

---

## 🛠️ Convenzioni del Repository

* **Standard di Naming:** File e cartelle seguono la convenzione `kebab-case` (`tutto-minuscolo-con-trattini`), privi di spazi o caratteri riservati per garantire massima compatibilità tra Windows, Linux e i visualizzatori Git/Obsidian.
* **Aggiornamento Continuo:** I cheat sheet e i playbook vengono periodicamente integrati e raffinati man mano che vengono completate nuove stanze di laboratorio.
