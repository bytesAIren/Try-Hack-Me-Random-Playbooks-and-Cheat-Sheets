# 🔓 RAG Sensitive Information Disclosure: Retrieval Abuse e Casi Reali

* **Dominio:** RAG Confidentiality, Indirect Prompt Injection, Model Extraction
* **Standard:** OWASP LLM01/LLM02/LLM10, MITRE ATLAS AML.T0024, GDPR, ISO/IEC 27001
* **Tipo:** teoria applicata e incident analysis

## 1. OWASP LLM02 nei sistemi RAG

La Sensitive Information Disclosure comprende PII, credenziali, documenti riservati, prompt, metadati, logiche proprietarie e dati di altri tenant restituiti tramite output, log o strumenti. Il leakage può avvenire senza exploit se la pipeline considera affidabili fonti, log o contesto.

È distinta dal poisoning: il poisoning altera ciò che viene recuperato, la disclosure espone ciò che non dovrebbe raggiungere l'utente; spesso i due attacchi sono concatenati.

## 2. Retrieval abuse

- inserire contenuti in fonti indicizzate per pilotare il contesto;
- sfruttare email HTML, pagine web o documenti con istruzioni indirette;
- usare query ripetute per ricostruire dati o comportamenti;
- osservare score, citazioni e differenze di risposta per inferire il corpus;
- sfruttare logging verboso e tracing per ottenere prompt e chunk riservati.

Il modello generativo non può sostituire ACL e DLP: il testo recuperato deve essere trattato come non fidato e delimitato strutturalmente.

## 3. Casi di studio

### EchoLeak — CVE-2025-32711

Dimostra il rischio di indirect prompt injection attraverso contenuti recuperati, come email HTML: il contenuto esterno può essere interpretato come istruzione e contribuire all'esfiltrazione. Mitigazioni: sanitizzazione, classificazione dei chunk, blocco del rendering automatico e controllo dei tool.

### Proof Pudding — CVE-2019-20634

L'output probing sistematico può ricostruire informazioni sul modello o sui dati osservando risposte, probabilità e differenze di comportamento. Ridurre la superficie: rate limiting, output minimizzato, niente confidence/ranking score pubblici e rilevamento di query seriali.

## 4. Logging e observability

Non registrare in chiaro prompt, documenti recuperati, token, PII o output sensibili se non indispensabile. Applicare redaction/NER, accesso separato ai log, retention breve, cifratura, audit e cancellazione coerente tra origine, cache, embedding, indice, backup e tracing.

## 5. Mapping e impatto

OWASP LLM02 copre la disclosure; LLM01 l'indirect prompt injection; LLM10 il model theft/unbounded consumption; LLM08 la debolezza di vector ed embedding. MITRE ATLAS AML.T0024 descrive l'esfiltrazione via inference API. Gli impatti includono data breach, violazione GDPR, cross-tenant access, perdita di proprietà intellettuale e compromissione di tool downstream.

