# ⚔️ AI Supply Chain Security: Attacchi, Recon e Incident Response

* **Dominio:** AI Supply Chain Security, Threat Hunting, Incident Reconstruction
* **Riferimenti:** MITRE ATLAS, OWASP LLM03, MITRE ATT&CK

## 1. Matrice dei vettori

| Vettore | Meccanismo | Indicatore |
| --- | --- | --- |
| Pickle / deserializzazione | `__reduce__`, import o callback durante il load | opcode globali, processi figli, egress inatteso |
| Dependency confusion | pacchetto pubblico con nome interno e versione alta | indice inatteso, maintainer sconosciuto, script di installazione |
| Typosquatting | nome quasi identico a pacchetto o namespace noto | differenze minime, metadata incoerenti, download improvvisi |
| Repository compromise | token o account usati per modificare artifact | commit/tag nuovi, file cambiati, token in log |
| API hosted | abuso provider, tool o output | prompt leakage, retention inattesa, azioni non autorizzate |

## 2. Ricognizione difensiva

In un ambiente autorizzato censire registry, bucket, notebook, endpoint di inferenza, vector DB, indici Python e provider esterni. Correlare inventario con DNS, log di autenticazione, egress e CI/CD. Dare priorità a servizi senza autenticazione, artifact scaricabili, token esposti e modelli caricati da origini non allowlisted.

## 3. Caso TryTrainMe: convergenza multi-vettore

La campagna combina tre livelli: un repository fittizio costruisce fiducia, un modello malevolo usa la deserializzazione come ingresso primario e una dipendenza pubblica omonima fornisce un ingresso di backup. Il principio operativo è che bloccare un solo canale non interrompe necessariamente la compromissione.

### Timeline sintetica

1. Vengono preparati namespace, modello e pacchetto con identità credibili.
2. Il modello viene scaricato e caricato nella pipeline.
3. Un'installazione ordinaria risolve anche la dipendenza pubblica.
4. Il SOC rileva beaconing o traffico HTTPS verso domini non previsti.

## 4. Risposta all'incidente

1. Isolare il runner o l'host senza distruggere le evidenze.
2. Revocare e ruotare token, credenziali e chiavi usate dalla pipeline.
3. Conservare hash, log, package metadata, process tree e connessioni di rete.
4. Identificare tutti gli artifact e le dipendenze ottenuti dalla stessa origine.
5. Ricostruire la provenienza, bloccare gli indicatori e riprodurre il deploy da fonti verificate.
6. Introdurre sandbox, lockfile, registry privati, firma degli artifact e controllo egress.

## 5. Regola operativa

La supply chain AI richiede difesa stratificata: verifica del repository, verifica del file, verifica della dipendenza, isolamento dell'esecuzione e monitoraggio dell'uscita di rete devono essere controlli indipendenti.

