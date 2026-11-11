# 🛡️ RAG Security: Difesa, Detection e Laboratorio Operativo

* **Dominio:** RAG Defense-in-Depth, Poisoning Detection, AI Governance
* **Riferimenti:** OWASP Top 10 for LLMs, NIST AI RMF, EU AI Act
* **Tipo:** pratica e hardening

## 1. Controlli all'ingestione

- allowlist delle fonti e autenticazione dei connettori;
- scansione malware, parser isolati e limiti di dimensione;
- normalizzazione, deduplicazione e provenance per documento/chunk;
- validazione di metadata, ACL, tenant e timestamp;
- rilevamento di istruzioni sospette e contenuti fuori dominio;
- approvazione o quarantine per fonti nuove e modifiche massive.

## 2. Controlli su embedding e retrieval

- namespace e filtri ACL applicati lato server;
- reranking e soglie di similarità calibrate su baseline;
- diversità delle fonti per evitare che copie duplicate dominino il contesto;
- limite al numero di chunk e alle istruzioni recuperate;
- marcatura esplicita del contenuto come non fidato;
- citazioni e tracciamento di documento, versione e score in ogni risposta.

## 3. Context guardrails e output

Il context builder deve separare istruzioni applicative e contenuto recuperato con delimitatori strutturali, senza considerare il testo recuperato una direttiva. Validare l'output con schema, policy e controlli DLP prima di mostrarlo o passarlo a tool. Azioni irreversibili richiedono minimo privilegio e conferma umana.

## 4. Detection stratificata

1. **Input:** anomalie di frequenza, duplicati, trigger, metadata e publisher.
2. **Ingestion:** variazioni improvvise di volume, cluster semantici anomali e fonti fuori allowlist.
3. **Index:** drift degli embedding, concentrazione eccessiva di un publisher, cambi di ranking.
4. **Retrieval:** score insoliti, ACL mismatch, documenti mai osservati e contesto fuori dominio.
5. **Output:** contraddizioni, leakage, rifiuti anomali, cambi comportamentali e tool call inattese.

Confrontare sempre con una baseline; nessun singolo segnale prova il poisoning.

## 5. Laboratorio concettuale

```python
from sklearn.metrics.pairwise import cosine_similarity

scores = cosine_similarity(query_vector, corpus_vectors)[0]
top_k = scores.argsort()[-5:][::-1]
for i in top_k:
    print(i, scores[i], metadata[i]["source"], metadata[i]["version"])
```

Ripetere la query prima e dopo l'aggiunta controllata di un documento, verificando ranking, provenance, ACL e risposta. Il test deve usare dati sintetici, ambiente isolato e nessuna credenziale reale.

## 6. Governance e risposta

Conservare owner, fonte, versione, hash, embedding model, policy e log di retrieval. In caso di sospetto: congelare reindicizzazione, isolare la fonte, preservare evidenze, revocare accessi, ricostruire l'indice da una baseline verificata e rieseguire test di regressione. Mappare i controlli su OWASP, NIST AI RMF ed eventuali obblighi di rischio e tracciabilità.

