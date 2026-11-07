# 🚩 CTF & Lab Writeups

Raccolta di walkthrough, writeup tecnici e risoluzioni pratiche di stanze e sfide di **TryHackMe**, Capture The Flag (CTF) e ambienti di laboratorio.

---

## 🗂️ Indice dei Writeup per Categoria

### 🤖 AI & LLM Security
* [**Escaping the Matrix (Prompt Injection & AI Jailbreak CTF)**](ai-security/escaping-the-matrix.md)  
  *Target:* Assistente terminale sandboxed con persona `Agent Smith`.  
  *Tecniche Chiave:* Navigazione del workflow narrativo interdipendente (🐇, 📞, 🚪), evasione del pattern *In-Context Refusal Reinforcement*, ricognizione dei vincoli interni e del database (`CLIENTS`, limit 3), bypass semantico tramite query su *VIP clients* per estrarre tutte e tre le flag (`THM{...}`).

---

## 🛠️ Metodologia di Documentazione dei Writeup

Ogni writeup è strutturato secondo standard di reportistica tecnica:
1. **Challenge Overview & Briefing:** Obiettivi, perimetro e meccanismi di difesa iniziali.
2. **Reconnaissance & Enumeration:** Analisi passiva/attiva, errori riscontrati e vettori scartati.
3. **Exploitation & Flag Extraction:** Dettaglio step-by-step dei comandi, interazioni e flag ottenute.
4. **Security Takeaways & Post-Mortem:** Lezioni apprese e fallimenti architetturali evidenziati dalla sfida.
