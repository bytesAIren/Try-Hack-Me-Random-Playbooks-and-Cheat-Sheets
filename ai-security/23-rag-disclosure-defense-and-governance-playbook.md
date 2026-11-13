# 🛡️ RAG Disclosure Defense: Controlli, Detection e Governance

* **Dominio:** RAG Defense-in-Depth, Data Controls, Red/Blue Team
* **Standard:** OWASP LLM01/LLM02/LLM07/LLM08, NIST AI RMF, EU AI Act Art. 9/10, GDPR
* **Tipo:** playbook pratico

## 1. Controlli all'ingestione

- allowlist delle fonti e autenticazione dei connettori;
- DLP/NER e redaction prima di chunking ed embedding;
- parser isolati, scansione malware e quarantine;
- metadata con owner, tenant, ACL, versione, hash e retention;
- deduplicazione, provenance e approvazione delle modifiche.

## 2. Retrieval filtering deterministico

```python
def retrieve(query, principal, vector_store, embedder, k=5):
    policy = policy_engine.allowed_scope(principal)
    query_vector = embedder.encode(query)
    # Il filtro ACL viene applicato prima della similarity search.
    return vector_store.search(
        vector=query_vector,
        top_k=min(k, 5),
        metadata_filter={"tenant": policy.tenant, "acl": {"$in": policy.groups}},
    )
```

Il codice è illustrativo: l'enforcement deve avvenire server-side, con identità verificata e test contro bypass di metadata, namespace e parametri HTTP.

## 3. Context, output e tool

- delimitare chiaramente istruzioni e dati recuperati;
- non eseguire automaticamente URL, HTML o istruzioni nei documenti;
- validare output con schema e policy;
- applicare DLP/redaction in uscita;
- non passare dati recuperati a shell, SQL o API senza autorizzazione separata;
- confermare azioni irreversibili con human-in-the-loop.

## 4. Logging e retention

Redigere PII e segreti, separare audit log da contenuto applicativo, limitare accessi e retention. Implementare la deletion chain: documento sorgente → cache → chunk → embedding → indice → backup → tracing. Verificare che revoche e cancellazioni siano effettive.

## 5. Detection e red/blue team

Monitorare query ripetute, variazioni anomale di top-k/score, cross-tenant mismatch, fonti mai viste, prompt injection nei chunk, output con segreti, token leakage, cambi comportamentali e accessi ai log. Il red team deve testare leakage, retrieval abuse, namespace bypass, logging exposure e model extraction; il blue team deve correlare questi segnali in SIEM e mantenere una baseline.

## 6. Governance e risposta

Applicare NIST AI RMF alle funzioni Govern/Map/Measure/Manage e documentare owner, DPIA, policy, test e incident response. In caso di disclosure: bloccare la fonte o l'endpoint, preservare log e hash, ruotare credenziali, revocare accessi, ricostruire indice e cache da una baseline verificata, notificare secondo GDPR e rieseguire test di regressione.

