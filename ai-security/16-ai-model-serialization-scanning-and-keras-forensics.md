# 🔍 AI Model Security: Serializzazione, Scanning e Keras Forensics

* **Dominio:** Secure Serialization, Static Analysis, Keras/HDF5 Forensics
* **Riferimenti:** MITRE ATLAS AML.T0010/AML.T0018, OWASP LLM03

## 1. Principio di sicurezza

Un modello può essere pericoloso in due momenti distinti: durante il caricamento, se il formato consente deserializzazione eseguibile, oppure durante l'inferenza, se l'architettura contiene logica malevola. SafeTensors riduce il primo rischio, ma non sostituisce la verifica di provenienza e architettura.

## 2. Hardening della serializzazione

- preferire **SafeTensors** per i soli tensori;
- in PyTorch usare `torch.load(..., weights_only=True)` quando compatibile;
- convertire i modelli legacy in ambiente isolato e verificare l'output;
- non usare `pickle.load()` su file non attendibili;
- trattare ONNX, GGUF e HDF5 come formati da validare, non come garanzia assoluta.

## 3. Scanning statico

```bash
fickling --check suspicious.pkl
modelscan scan suspicious.pkl
modelscan scan suspicious.h5
python3 -c "import h5py; f=h5py.File('model.h5','r'); print(list(f.keys()))"
```

Fickling aiuta a leggere il bytecode Pickle; ModelScan rileva indicatori su più formati; `h5py` permette di confrontare struttura e configurazione HDF5 senza eseguire il modello. Conservare sempre hash e report.

## 4. Keras Lambda Layer

I Lambda layer possono contenere funzioni Python custom e introdurre comportamento inatteso all'inference time. Confrontare un modello candidato con una baseline: numero e nomi dei layer, configurazione, funzioni serializzate e differenze nel grafo. Un layer `Lambda` non è automaticamente malevolo, ma richiede giustificazione e revisione.

## 5. Limiti e decisione

Lo scanning statico può non rilevare codice offuscato, backdoor nei pesi o comportamenti attivati solo da input specifici. La decisione deve combinare scanning, provenance, sandbox, test funzionali e monitoraggio runtime.

