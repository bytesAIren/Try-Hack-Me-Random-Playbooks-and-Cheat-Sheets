# 🛡️ AI Supply Chain Governance: Integrità, Dipendenze e SBOM

* **Dominio:** Model Integrity, Dependency Hardening, SBOM, Model Acquisition
* **Riferimenti:** MITRE ATLAS, OWASP LLM03, NIST AI RMF, SPDX/CycloneDX

## 1. Model Acquisition Framework

1. definire origini e registry autorizzati;
2. valutare model card, publisher, licenza, dataset e dipendenze;
3. scaricare in quarantine e calcolare SHA-256;
4. verificare firma, digest, commit e provenienza;
5. scansionare e valutare il modello in sandbox;
6. approvare una versione immutabile e monitorarne l'uso.

Un hash conferma che il file è quello atteso, non che il file sia benigno: servono sia integrità sia provenance.

## 2. LoRA e model merging

Gli adattatori LoRA e i servizi di merging ereditano la fiducia dei modelli base e possono alterarne il comportamento. Registrare base model, adapter, dataset, commit e configurazione; impedire merge da origini non verificate.

## 3. Dependency hardening

```text
version pinning → lockfile con hash → private index → pip-audit → SBOM → review
```

Usare versioni esatte, lockfile verificabili e un indice privato come origine primaria. Evitare risoluzioni ambigue tra registry pubblico e privato per ridurre dependency confusion.

```bash
pip-audit -r requirements.txt
syft dir:. -o cyclonedx-json=sbom.json
```

## 4. SBOM per sistemi AI

Oltre ai pacchetti includere modello, digest, dataset, tokenizer, runtime, immagini container, plugin, adapter LoRA e licenze. Collegare ogni componente a owner, origine, versione e CVE; rigenerare l'SBOM a ogni build o cambio di artifact.

## 5. Controlli di accettazione

- [ ] hash e firma verificati;
- [ ] publisher e model card valutati;
- [ ] dipendenze bloccate e scansionate;
- [ ] SBOM archiviato;
- [ ] modello testato in sandbox;
- [ ] rollback e revoca della versione disponibili.

