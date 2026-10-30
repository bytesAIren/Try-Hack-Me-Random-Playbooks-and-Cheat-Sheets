# 🧠 Neural Network Fundamentals: Feedforward & Layer Abstraction

Un modello di classificazione basato su reti neurali feedforward elabora l'informazione trasformando dati grezzi a bassa dimensionalità in rappresentazioni semantiche ad alto livello attraverso una sequenza di layer.

---

## 🏗️ Architettura a 3 Livelli (Feedforward Flow)

[ Input Layer ] ────────▶ [ Hidden Layer(s) ] ────────▶ [ Output Layer ]
(Raw Features) (Pattern Abstraction) (Classification / Label)
code Code

### 1. Input Layer (Caratteristiche Grezze)
* **Ruolo:** Riceve le misurazioni dirette dell'ambiente (feature numeriche, pixel, token o attributi categorici codificati).
* **Livello di astrazione:** Minimo (zero contesto semantico).
* **Esempio del lab (Golden Retriever):**
  * `Features: [golden fur, four legs, floppy ears, wagging tail]`
* **Esempio in Cyber/SOC (Network Traffic):**
  * `Features: [source_port: 54331, dest_port: 8067, packet_size: 512, protocol: TCP]`

---

### 2. Hidden Layer(s) (Estrazione di Pattern e Astrazione)
* **Ruolo:** Applica combinazioni lineari pesate e funzioni di attivazione non lineari (es. ReLU, Sigmoid) per identificare correlazioni tra le feature di ingresso.
* **Livello di astrazione:** Intermedio (pattern compositi non direttamente misurabili dai singoli sensori).
* **Esempio del lab:**
  * Combinando pelo, 4 zampe e coda $\rightarrow$ emerge il pattern: `[mammal, domestic pet, quadruped, friendly behaviour]`.
* **Esempio in Cyber/SOC:**
  * Combinando porta di destinazione non standard, traffico outbound e assenza di resolver DNS noto $\rightarrow$ emerge il pattern: `[beaconing periodico, probabile canale C2 non cifrato]`.

---

### 3. Output Layer (Classificazione Finale)
* **Ruolo:** Assegna una distribuzione di probabilità (spesso tramite funzione *Softmax*) su un insieme predefinito di classi discrete (*labels*).
* **Livello di astrazione:** Massimo (decisione categoriale).
* **Esempio del lab:**
  * `Classification: Golden Retriever` (decisione finale del nodo di output).
* **Esempio in Cyber/SOC:**
  * `Classification: Malicious Traffic / C2 Beacon (Confidence: 0.94)`.

---

## 🛡️ Rilevanza per l'AI Security (Perché serve a un security analyst?)

Capire questo flusso è il prerequisito per analizzare i vettori di attacco specifici per il Machine Learning:

| Vettore d'Attacco | Come sfrutta i Layer | Obiettivo dell'Attaccante |
| :--- | :--- | :--- |
| **Adversarial Perturbation (Evasion)** | Modifica impercettibile delle feature nell'**Input Layer** (es. rumore invisibile sui pixel o byte dummy nei pacchetti). | Indurre l'**Hidden Layer** a costruire una rappresentazione errata, portando l'**Output Layer** a classificare un file malevolo come benigno. |
| **Data Poisoning** | Iniezione di esempi con label alterate o backdoor trigger durante l'addestramento. | Forzare la rete ad associare un pattern specifico (es. una stringa fissa) a una classe benigna garantita. |
| **Model Inversion / Extraction** | Analisi ripetuta degli output (confidenze dell'**Output Layer**) partendo da input controllati. | Ricostruire a ritroso le feature sensibili utilizzate nel dataset di training. |
