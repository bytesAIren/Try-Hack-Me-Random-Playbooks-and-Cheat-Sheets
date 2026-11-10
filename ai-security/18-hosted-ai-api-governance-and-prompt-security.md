# 🌐 Hosted AI API: Governance e Sicurezza dei Prompt

* **Dominio:** Hosted AI Security, Provider Governance, System Prompt Security
* **Riferimenti:** MITRE ATLAS AML.T0010, OWASP LLM03/LLM07

## 1. Modello di responsabilità

Con un'API hosted il consumatore non può ispezionare pesi, training data o runtime del provider. Il rischio si sposta su due piani: due diligence del fornitore e sicurezza dell'applicazione che invia prompt, riceve output e concede tool.

## 2. Due diligence del provider

Verificare retention e uso dei dati, isolamento tenant, regioni, logging, incident response, versioning del modello, SLA, subprocessor, controlli di accesso e possibilità di opt-out dal training. Documentare modello, endpoint, policy e autorizzazioni per ogni ambiente.

## 3. Prompt come artefatti di supply chain

System prompt, template, tool schema e policy sono componenti versionabili: proteggerli con repository privati, review, approvazione e controllo degli accessi. Non inserire segreti nel prompt e non considerare le istruzioni una barriera di sicurezza deterministica.

## 4. Controlli applicativi

- classificare e minimizzare i dati inviati;
- redigere PII e segreti prima della richiesta;
- validare l'output con schema rigido;
- applicare allowlist di tool e minimo privilegio;
- richiedere conferma umana per azioni irreversibili;
- limitare rate, costi, contesto e traffico in uscita;
- monitorare prompt injection, leakage, anomalie e cambiamenti di comportamento.

## 5. Valutazione in sandbox

Testare ogni provider o versione con casi benigni, prompt injection dirette/indirette, richieste di dati sensibili, tool abuse e output non conforme. Ripetere i test dopo cambi di modello, system prompt, policy o integrazione e conservare gli esiti come evidenza di governance.

