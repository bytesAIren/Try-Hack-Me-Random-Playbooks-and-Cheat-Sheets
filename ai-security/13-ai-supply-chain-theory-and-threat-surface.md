# 📦 AI Supply Chain Security: Teoria e Superficie d'Attacco

* **Dominio:** AI Supply Chain Security, MLOps Governance, Hosted AI
* **Riferimenti:** MITRE ATLAS, OWASP Top 10 for LLMs, NIST AI RMF
* **Scopo:** comprendere i componenti della catena di fornitura AI e i rischi dei modelli scaricati, dei pacchetti Python, dei repository e delle API gestite.

## 1. Perimetro della supply chain AI

Una pipeline AI estende la supply chain software tradizionale: oltre al codice e alle dipendenze comprende modelli, pesi, dataset, tokenizer, artefatti MLOps, notebook, registri e servizi di inferenza.

| Livello | Asset | Rischio principale |
| --- | --- | --- |
| Model | `.pkl`, `.pt`, `.onnx`, `.safetensors`, `.gguf` | codice durante il caricamento, backdoor, pesi manipolati |
| Dependency | PyPI, wheel, container, framework | dependency confusion, typosquatting, maintainer compromise |
| Data | dataset, RAG, tokenizer, fine-tuning | poisoning, bias intenzionale, esfiltrazione |
| Repository | Hugging Face, registri interni, Git | namespace impersonation, token esposti, artifact sostituiti |
| Infrastructure/API | MLflow, Jupyter, vector DB, provider hosted | accesso anonimo, metadata leakage, abuso di privilegi |

## 2. Download dei modelli vs API gestite

Con il download il consumatore entra in possesso di un file e deve verificarne formato, provenienza e comportamento. Con un'API hosted il file non è esposto, ma la fiducia si sposta sul provider: diventano centrali retention dei prompt, isolamento dei tenant, logging, endpoint, plugin, system prompt leakage e output non validato.

La scelta non elimina il rischio: modifica il punto di controllo e il modello di responsabilità condivisa.

## 3. Formati e rischio di deserializzazione

- **Pickle / `.pkl` / `.pt` legacy:** può invocare codice arbitrario in fase di deserializzazione; non va trattato come semplice dato.
- **`safetensors`:** preferibile per i soli tensori, con caricamento più prevedibile.
- **ONNX:** richiede controllo del grafo, degli operatori e del runtime.
- **GGUF:** riduce alcuni rischi di serializzazione Python, ma non sostituisce la verifica della provenienza e dell'architettura.
- **Keras Lambda e oggetti custom:** possono introdurre esecuzione o comportamento non desiderato anche senza un file Pickle evidente.

## 4. Vettori ricorrenti

1. **Dependency confusion:** un nome interno viene registrato su un indice pubblico con una versione apparentemente più nuova.
2. **Typosquatting:** un pacchetto o namespace imita quello legittimo.
3. **Repository compromise:** token rubati o account compromessi alterano un modello già considerato affidabile.
4. **API e MLOps esposti:** MLflow, notebook, registri e vector database rivelano artefatti o credenziali.
5. **Campagne multi-vettore:** repository, modello e dipendenze forniscono percorsi ridondanti verso lo stesso host.

## 5. Controlli architetturali

- usare registry privati e allowlist di origini, digest e commit;
- firmare gli artefatti e verificare hash, provenance e model card;
- bloccare i pacchetti interni sugli indici pubblici e usare lockfile;
- isolare il caricamento dei modelli in sandbox con egress limitato;
- applicare minimo privilegio a CI/CD, notebook, API e provider;
- validare schema e output prima di passarli a shell, SQL, browser o API distruttive;
- registrare download, caricamenti, richieste API e anomalie di rete.

