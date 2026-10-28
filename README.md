# 🛡️ The Operator's Playbook: Metodologia di Testing & Lab

```
[ Recon / Scansione ] ➔ [ Surface Mapping ] ➔ [ Initial Foothold ] ➔ [ Post-Exploitation ] ➔ [ PrivEsc ] ➔ [ Documentazione ]
```

---

## FASE 1: Ricognizione e Scansione di Rete (Discovery)

L'obiettivo è identificare le porte aperte e le versioni esatte dei servizi in esecuzione, evitando scansioni rumorose o incomplete.

### Checklist Operativa:
- [ ] **Ping Check iniziale:** Verificare se il target risponde all'ICMP (nota: molti host Windows bloccano il ping di default).
- [ ] **Scansione Porte Completa (TCP):** Identificare tutte le 65.535 porte.
- [ ] **Scansione Mirata sui Servizi (Service Version & Script Scan):** Identificare versioni esatte (`-sV`) ed eseguire gli script di default (`-sC`).
- [ ] **Scansione UDP rapida:** Limitata alle porte UDP critiche (SNMP `161`, DNS `53`, TFTP `69`).

### Sintassi di Riferimento (Nmap):
```bash
# 1. Scansione veloce di tutte le porte TCP (salva l'output in formato grep/file)
nmap -p- --min-rate 1000 -T4 -oN all_ports.txt <TARGET_IP>

# 2. Scansione approfondita solo sulle porte scoperte (es. 22, 80, 445)
nmap -p <PORTE_APERTE> -sC -sV -oN targeted_scan.txt <TARGET_IP>
```

---

## FASE 2: Mappatura della Superficie d'Attacco (Surface Mapping)

La regola d'oro: **non tirare a indovinare.** Ogni porta aperta impone verifiche precise.

| Porta / Servizio | Domande di Verifica Metodologica | Tool di Analisi Consigliati |
| :--- | :--- | :--- |
| **80 / 443 / 8080 (HTTP/S)** | Qual è il web server? Che framework è in uso (PHP, Node/Next.js, Python)? Ci sono directory/API nascoste? Ci sono moduli di login o upload? | Browser (F12), Burp Suite, Wappalyzer, Feroxbuster / Gobuster, Nikto |
| **21 (FTP)** | È abilitato l'accesso anonimo (`anonymous:anonymous`)? Ci sono file di configurazione o backup scaricabili? | Client `ftp`, Nmap script `ftp-anon` |
| **22 (SSH)** | È ammessa l'autenticazione con password debole? Le versioni di OpenSSH sono soggette a falle note? | Client `ssh`, Nmap |
| **445 / 139 (SMB)** | Esistono share accessibili senza autenticazione (Null Session)? Ci sono file `.txt`, `.bak` o chiavi SSH nelle cartelle condivise? | `smbclient -L //<IP>/ -N`, `cme smb` / `NetExec`, `enum4linux-ng` |
| **161 (SNMP)** | Le stringhe di community di default (`public`, `private`) sono attive? Rivelano processi, utenti o interfacce? | `snmpwalk -v2c -c public <IP>` |
| **88 / 389 (Kerberos / LDAP)** | (Ambiente Active Directory): È un Domain Controller? È possibile fare Kerberoasting o AS-REP Roasting? | `NetExec`, `ldapsearch`, `BloodHound` |

---

## FASE 3: Penetrazione Iniziale (Initial Foothold)

Una volta isolato il punto debole, si valuta il vettore di ingresso. Il percorso decisionale segue questa gerarchia:

```
Esiste un'autenticazione predefinita o debole? (admin/admin, guest, anonymous)
  ├── SÌ ➔ Verifica accesso e privilegi.
  └── NO ➔ 
       ├── C'è un'applicazione Web? ➔ Verifica OWASP Top 10:
       │    ├── Upload insicuro (Webshell)
       │    ├── Command Injection (RCE)
       │    ├── SQL Injection (Estrazione credenziali o lettura file)
       │    └── Auth Bypass / IDOR
       └── C'è un servizio esposto con versione vulnerabile nota?
            └── Cerca PoC pubblico (Exploit-DB, GitHub, CVE database)
```

### Regole Operative per il Foothold:
1. **Comprendere il Payload:** Non lanciare mai uno script di exploit scaricato da GitHub senza averne letto il sorgente. Verifica cosa invia, se richiede parametri particolari (come l'encoding Base64 per evitare errori di quoting) e dove punta la connessione.
2. **Stabilizzazione della Shell:** Appena ottenuto l'accesso (reverse shell), il primo passo è stabilizzare il terminale per evitare crash accidentali (es. premendo `Ctrl+C`).
   ```bash
   # Su target Linux (per ottenere una TTY completa):
   python3 -c 'import pty; pty.spawn("/bin/bash")'
   # Poi premi Ctrl+Z per sospendere la shell, e nel tuo terminale locale scrivi:
   stty raw -echo; fg
   export TERM=xterm
   ```

---

## FASE 4: Post-Exploitation & Privilege Escalation (PrivEsc)

Sei dentro come utente a basso privilegio (es. `www-data`, `daniel`, `s.reyes`). Ora devi ottenere il controllo totale (**Root** su Linux, **SYSTEM** su Windows).

### Checklist Linux:
1. **Identità & Gruppi:**
   ```bash
   id
   whoami
   ```
2. **Permessi Sudo (La prima verifica obbligatoria):**
   ```bash
   sudo -l -n
   ```
   *Se trovi comandi consentiti senza password:* Consulta immediatamente [GTFOBins](https://gtfobins.github.io/) per identificare se l'eseguibile ammette breakout verso la shell.
3. **Binari con SUID bit (Eseguiti con permessi del proprietario):**
   ```bash
   find / -perm -u=s -type f 2>/dev/null
   ```
4. **Scheduled Tasks / Cron Jobs:**
   Verificare script eseguiti periodicamente da root che hanno permessi di scrittura aperti per gli utenti normali:
   ```bash
   cat /etc/crontab
   ls -la /etc/cron.*
   ```
5. **File Interni e Credenziali dimenticate:**
   Cercare password nei file di configurazione web (`wp-config.php`, `.env`, file `.json` o stringhe di connessione database).
   ```bash
   grep -rn "password" /var/www/ 2>/dev/null
   ```

---

### Checklist Windows:
1. **Identità e Privilegi dell'Account:**
   ```cmd
   whoami /priv
   whoami /groups
   ```
   *Attenzione a privilegi critici come:* `SeImpersonatePrivilege` (vulnerabile a tecniche di token abuse stile *Potato*).
2. **Servizi di Sistema & Unquoted Service Paths:**
   Verificare servizi con percorsi privi di virgolette o cartelle con permessi di scrittura deboli.
3. **Credenziali in chiaro memorizzate:**
   Ispezionare registri, file `Unattend.xml` o il Windows Credential Vault (`vaultcmd.exe`).
4. **Scheduled Tasks attive:**
   ```cmd
   schtasks /query /fo LIST /v
   ```

---

## FASE 5: Movimento Laterale & Pivoting (Lab Multi-Host / Active Directory)

Nelle reti complesse, la prima macchina compromessa è solo una testa di ponte (*Beachhead*). Per raggiungere i server interni:

1. **Internal Network Discovery:**
   Ispezionare la tabella di routing e la cache ARP dell'host per scoprire subnet e computer adiacenti:
   ```bash
   # Su Linux:
   ip route; arp -a
   # Su Windows:
   route print; arp -a
   ```
2. **Pivoting / Tunnelling:**
   Usare tool di inoltro del traffico (come **Chisel**, **Ligolo-ng** o tunnel SSH con `-D`) per instradare gli strumenti d'attacco della tua macchina (es. Nmap, proxychains, Burp Suite) direttamente dentro la rete interna del laboratorio.

---

## FASE 6: Chiusura & Documentazione (Reporting)

Che sia un CTF su HTB o un test formale, la prova si conclude con un riassunto strutturato:

1. **Executive Summary:** Cosa è successo, qual era il livello di rischio per il business, quale risorsa è stata compromessa.
2. **Kill Chain Narrative:** La timeline cronologica dettagliata passo dopo passo.
3. **Tabella IOCs (Indicatori di Compromissione):** File creati, hash, indirizzi IP, chiavi di registro.
4. **Remediation Plan (Le Contromisure):**
   * Come correggere la causa radice (aggiornamento software, patching).
   * Come applicare l'hardening (principio del minimo privilegio, rimozione di regole sudo permissive, disabilitazione porte non necessarie).

---
