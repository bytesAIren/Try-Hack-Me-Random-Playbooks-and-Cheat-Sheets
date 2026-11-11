# 📚 RAG Security: Architettura, Poisoning e Retrieval Abuse

* **Dominio:** Retrieval-Augmented Generation, Data Integrity, Vector Database Security
* **Riferimenti:** OWASP Top 10 for LLMs, NIST AI RMF, MITRE ATLAS
* **Tipo:** teoria e threat modeling

## 1. Architettura e confini di fiducia

Un sistema RAG collega ingestion pipeline, parser, chunker, embedding model, vector database, retriever, prompt builder e LLM. Ogni passaggio è un confine di sicurezza: il contenuto recuperato è dati non fidati, anche quando viene presentato al modello insieme alle istruzioni applicative.

```text
fonti → ingestion/parser → chunking → embedding → vector DB
                                      ↓
query → embedding → retrieval/reranking → context guardrail → LLM → output/tool
```

I punti più delicati sono ingestion automatizzata, metadata e ACL, corpus vettoriale, ranking e composizione del contesto.

## 2. Data e model poisoning

Il poisoning può avvenire nel dataset di training, nel fine-tuning, nella memoria RAG, nei documenti sorgente o direttamente nel corpus indicizzato. Non è necessario modificare i pesi: basta fare in modo che il retriever selezioni contenuti manipolati o che il modello apprenda una correlazione indesiderata.

Gli effetti possono essere evidenti, come risposte palesemente errate, oppure subdoli: bias, deriva comportamentale, trigger rari, omissioni selettive e degradazione progressiva. I dati avvelenati persistono perché vengono copiati in cache, embedding, indici, dataset derivati e prompt di valutazione.

## 3. Embedding e corpus poisoning

La ricerca per similarità seleziona i documenti più vicini alla query nello spazio vettoriale. Un attaccante può duplicare contenuti, manipolare testo o metadata, ottimizzare parole chiave e creare documenti semanticamente vicini per aumentare la probabilità di retrieval. I documenti legittimi restano intatti ma diventano meno competitivi.

Il rischio è quindi diverso dal prompt injection diretto: l'istruzione malevola entra indirettamente nel contesto attraverso una fonte apparentemente pertinente.

## 4. Retrieval abuse

- **Attivo:** l'attaccante pubblica o modifica contenuti per influenzare il retrieval.
- **Passivo:** sfrutta fonti obsolete, permissive o già compromesse per ottenere risposte errate o dati fuori perimetro.
- **Metadata/ACL abuse:** altera filtri, namespace o permessi per far recuperare documenti di un altro tenant.
- **Context smuggling:** nasconde istruzioni nei documenti, commenti, HTML o metadati.

Il modello non può distinguere in modo affidabile istruzioni e dati recuperati: la separazione deve essere realizzata dall'applicazione, dal retriever e dai controlli di output.

## 5. Impatti e threat model

Gli impatti includono misinformation, data leakage, violazione di tenant isolation, bypass dei guardrail, tool abuse, deriva del comportamento e perdita di auditabilità. Mappare ogni fonte, owner, ACL, trasformazione, indice e consumer; identificare chi può scrivere, reindicizzare, cambiare embedding model o modificare il prompt builder.

