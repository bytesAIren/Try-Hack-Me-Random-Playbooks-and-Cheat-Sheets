# 🔐 RAG Retrieval Security: Confini di Sicurezza e Privacy degli Embedding

* **Dominio:** RAG Architecture Security, Multi-Tenant Isolation, Vector Database Privacy
* **Standard:** OWASP LLM02/LLM08, MITRE ATLAS AML.T0024, GDPR, CWE-284/CWE-287/CWE-862
* **Tipo:** teoria e threat modeling

## 1. Il retrieval è un confine di sicurezza

Il retrieval non è solo una funzione di ricerca: decide quali dati raggiungono il modello. Il controllo degli accessi deve quindi essere applicato prima della similarity search, non dopo il recupero e non tramite il System Prompt.

```text
utente → autenticazione → policy/tenant → metadata pre-filter
       → similarity search → reranking → context builder → LLM
```

Un singolo indice condiviso senza enforcement deterministico può causare cross-tenant leakage anche quando i documenti hanno metadata corretti.

## 2. Rischi di configurazione

- `top-k` eccessivo amplia il contesto e aumenta la probabilità di leakage;
- filtri metadata mancanti o applicati post-query lasciano entrare candidati non autorizzati;
- namespace logici non equivalgono a isolamento reale;
- embedding obsoleti sopravvivono a revoche e cancellazioni;
- score, ranking e metadata esposti facilitano probing e model extraction.

## 3. Pattern di segmentazione

Preferire, in ordine di isolamento, database/cluster separati, collection o namespace per tenant e pre-filtering rigoroso su un indice condiviso. Le ACL devono essere calcolate da identità autenticata e policy server-side; mai da parametri controllati dall'utente.

## 4. Privacy nello spazio vettoriale

Gli embedding possono rivelare similarità, appartenenza a un corpus e caratteristiche del testo originale. Considerare embedding inversion, membership inference, query probing e correlazione tra metadata. Minimizzare dati sensibili prima dell'embedding, cifrare a riposo/in transito, limitare query e non restituire score interni agli utenti.

## 5. Casi tecnici

### AnythingLLM — CVE-2024-3033

Endpoint API esposti senza autenticazione/autorizzazione hanno consentito di raggiungere workspace e vector store. La lezione è che la separazione logica dei workspace non protegge nulla se gli endpoint non impongono CWE-862 (Missing Authorization).

### Milvus Proxy — CVE-2025-64513

Un controllo di autenticazione errato si fidava dell'header controllabile `sourceId`. La falsificazione dell'header permetteva di superare il proxy, dimostrando il rischio CWE-287 (Improper Authentication) su un gateway di retrieval.

## 6. Threat model

Identificare per ogni query identità, tenant, policy, namespace, filtri, documenti candidati, score e output. Collegare le evidenze a OWASP LLM02 (Sensitive Information Disclosure), LLM08 (Vector and Embedding Weaknesses) e MITRE ATLAS AML.T0024 (Exfiltration via ML Inference API).

