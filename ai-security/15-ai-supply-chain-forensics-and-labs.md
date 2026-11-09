# 🧪 AI Supply Chain Security: Forensics e Laboratorio Pratico

* **Dominio:** AI Forensics, Model Safety Audit, Package Auditing
* **Scopo:** ispezionare artifact sospetti senza eseguirli e verificare dipendenze, namespace e metadati.

## 1. Ispezione sicura di un Pickle

Operare in una macchina isolata, senza credenziali e con egress bloccato. Non usare `pickle.load()` sul campione.

```bash
sha256sum suspicious.pkl
file suspicious.pkl
python3 -m pickletools suspicious.pkl
python3 -m pickletools suspicious.pkl | grep -E 'GLOBAL|STACK_GLOBAL|REDUCE|BUILD|INST|OBJ'
strings -n 8 suspicious.pkl | less
```

Gli opcode che importano globali o ricostruiscono oggetti meritano revisione. Cercare riferimenti a `os`, `subprocess`, `socket`, `requests`, shell e callback. L'output di `pickletools` è disassembly: non è un caricamento del modello.

## 2. Audit di pacchetti Python

```bash
python3 -m pip index versions PACKAGE --index-url https://pypi.org/simple
python3 -m pip download --no-deps PACKAGE==VERSION -d ./quarantine
python3 -m zipfile -l ./quarantine/*.whl
unzip -l ./quarantine/*.whl
grep -RniE 'subprocess|os\.system|socket|requests|curl|wget|base64' ./quarantine
```

Confrontare nome, owner, homepage, release history, hash e indice configurato con la dipendenza prevista. Non installare il pacchetto sospetto nel sistema dell'analista; se serve esecuzione dinamica, usare un laboratorio effimero senza segreti.

## 3. Verifica di repository e namespace

- controllare ortografia esatta di organizzazione e modello;
- confrontare card, commit, tag, digest e data di pubblicazione;
- verificare se il publisher è quello atteso e se esistono release improvvise;
- cercare token nei log CI, notebook e configurazioni;
- scaricare solo tramite allowlist e conservare hash prima dell'uso.

## 4. Checklist finale

- [ ] Provenienza e hash documentati
- [ ] Formato e loader approvati
- [ ] Opcode o codice di installazione revisionati
- [ ] Dipendenze risolte da registry autorizzato
- [ ] Artifact testato in sandbox
- [ ] Egress e privilegi minimi applicati
- [ ] Evidenze e decisione di accettazione del rischio conservate

