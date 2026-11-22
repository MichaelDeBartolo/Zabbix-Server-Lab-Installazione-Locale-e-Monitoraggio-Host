# Laboratorio di Monitoraggio con Zabbix 7.0 su Linux — Guida Passo Passo

Guida pratica di **Infrastructure & Network Monitoring** in locale su Linux (Ubuntu) con **Zabbix 7.0**, **MariaDB**, **Apache** e **Zabbix Agent**, applicata al monitoraggio di un dispositivo sulla rete Wi-Fi locale (es. smartphone via ICMP Ping).  
L'obiettivo di questo laboratorio è mostrare come installare e configurare Zabbix, collegarlo al database, accedere all'interfaccia web, censire un host di rete e visualizzare disponibilità e latenza in dashboard.

> **Nota su credenziali e sicurezza:** Le credenziali riportate (`Admin` / `zabbix` per il primo accesso web e `passwordtest` come password di esempio per il database) sono valori dimostrativi/predefiniti per un laboratorio locale. Cambia sempre le password di default e non esporre il frontend Zabbix su reti non attendibili senza adeguate misure di sicurezza.

---

## Cosa fa in breve Zabbix (e perché si parte da qui)

Immagina di dover controllare in tempo reale se un dispositivo o un servizio è raggiungibile, quanto impiega a rispondere e se ci sono cali di prestazione o disconnessioni:
* **Zabbix Server & Agent** raccolgono e valutano metriche e controlli di stato.
* **MariaDB** conserva configurazioni, dati storici, eventi e andamenti nel tempo.
* **Frontend Web (Apache + PHP)** permette di consultare grafici, stati e dashboard dal browser.

Partire con un'installazione locale su un singolo PC consente di capire come collaborano server, database, web server e controlli di rete senza dover gestire subito un'infrastruttura distribuita.

---

## Architettura e Ambiente di Test

Quando Zabbix gira in locale, l'intera infrastruttura si appoggia sulle risorse della macchina ospitante:

| Componente / Risorsa | Ruolo | Dettagli nel laboratorio |
| :--- | :--- | :--- |
| **Sistema Operativo** | Host locale | Ubuntu (es. Ubuntu `24.04`) con permessi `sudo` |
| **Zabbix 7.0** | Motore di monitoraggio | `zabbix-server-mysql`, `zabbix-frontend-php`, `zabbix-apache-conf`, `zabbix-agent` |
| **Database** | Storage e dati storici | MariaDB/MySQL per configurazioni, cronologia metriche e grafici su disco |
| **Calcolo (CPU / RAM)** | Elaborazione ed esposizione | Server Zabbix, agente locale e Apache usano CPU e RAM dell'host |
| **Target di rete** | Dispositivo monitorato | Smartphone (es. Samsung S24) sulla stessa rete Wi-Fi locale via ICMP Ping |

### Schema dell'architettura locale

```text
┌──────────────────────────────────────────────────────────┐
│                     PC Host (Ubuntu)                     │
│                                                          │
│  ┌────────────────┐    ┌─────────────┐    ┌───────────┐  │
│  │ Zabbix Server  │◄──►│  MariaDB    │    │  Apache   │  │
│  │ & Zabbix Agent │    │ (Storage)   │    │ (Web UI)  │  │
│  └───────┬────────┘    └─────────────┘    └─────▲─────┘  │
└──────────┼──────────────────────────────────────┼────────┘
           │ ICMP Ping (Wi-Fi locale)             │ http://localhost/zabbix
           ▼                                      ▼
  [Smartphone / Host]                        [Browser]
```

---

## Passo 1 — Installazione dei Pacchetti Zabbix 7.0

Aggiungi il repository ufficiale Zabbix 7.0 per Ubuntu e installa server, frontend web, configurazione Apache e agente locale:

```bash
wget [https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_7.0-1+ubuntu24.04_all.deb](https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_7.0-1+ubuntu24.04_all.deb)
sudo dpkg -i zabbix-release_7.0-1+ubuntu24.04_all.deb
sudo apt update
sudo apt install -y zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-agent
```

### Verifica del passaggio
Controlla che i pacchetti siano installati senza errori e che i servizi risultino presenti sul sistema.

---
## Passo 2 — Configurazione del Database e del Server

Dopo aver predisposto il database su MariaDB e importato lo schema iniziale di Zabbix, imposta la password del database nel file `/etc/zabbix/zabbix_server.conf`:

```ini
### Option: DBPassword
#       Database password.
#       Comment this line if no password is used.

DBPassword=passwordtest
```

*(Nota: assicurati che il valore di `DBPassword` corrisponda esattamente alla password impostata sull'utente del database MariaDB e che il file abbia permessi adeguati.)*

### Avvio e abilitazione dei servizi al boot
Avvia `zabbix-server`, `zabbix-agent` e `apache2` e abilitali all'avvio automatico:

```bash
sudo systemctl restart zabbix-server zabbix-agent apache2
sudo systemctl enable zabbix-server zabbix-agent apache2
```

### Verifica del passaggio
Controlla che i tre servizi risultino attivi (`active (running)`) con `systemctl status zabbix-server zabbix-agent apache2`.

---

## Passo 3 — Accesso al Frontend Web

Apri il browser e raggiungi l'interfaccia web locale:

```text
http://localhost/zabbix
```

Completa l'eventuale procedura guidata iniziale (se non già configurata) e accedi con le credenziali predefinite:
* **Username:** `Admin` (con la `A` maiuscola)
* **Password:** `zabbix`

---

## Passo 4 — Censimento di un Host sulla Rete Locale (Esempio: Samsung S24)

Per testare il monitoraggio di rete senza installare un agente sul telefono, puoi configurare lo smartphone come host sulla stessa rete Wi-Fi locale e verificarne la raggiungibilità tramite ICMP.

### 4.1 Individuare l'indirizzo IP nella sottorete locale (opzionale)
Se non conosci l'IP del dispositivo sulla rete locale, puoi installare ed eseguire `arp-scan`:

```bash
sudo apt install -y arp-scan
sudo arp-scan --localnet
```

### 4.2 Creare l'host in Zabbix
Nel frontend web di Zabbix, vai su **Data collection → Hosts → Create host** e imposta:
* **Host name:** `Samsung s24`
* **Host groups:** `Discovered hosts` (gruppo obbligatorio per il salvataggio)
* **Interfaces:** aggiungi l'interfaccia indicando l'indirizzo IP del telefono sulla rete Wi-Fi e lasciando selezionato **IP**
* **Templates:** associa il template **ICMP Ping** (presente in `Templates/Modules`)

### Cosa misura il template ICMP Ping
* **Raggiungibilità del dispositivo** (stato online/offline).
* **Latenza di risposta Wi-Fi** (tempo di andata e ritorno in millisecondi).
* **Percentuale di pacchetti persi** (`packet loss`).

---
## Passo 5 — Visualizzazione Dati e Personalizzazione della Dashboard

Per avere a colpo d'occhio lo stato del dispositivo direttamente dalla **Dashboard** principale (**Global view**), puoi aggiungere due widget:

* **Host availability:** mostra un riquadro colorato che indica se l'host / il gruppo di dispositivi è disponibile o irraggiungibile.
* **Graph (classic):** impostato come **Simple graph** collegato alla metrica **ICMP response time** dell'host `Samsung s24`, così da visualizzare l'andamento della latenza nel tempo.

### Come leggere il comportamento in dashboard
Quando lo smartphone si disconnette dalla rete Wi-Fi o non risponde ai ping, la metrica di risposta si interrompe e la dashboard evidenzia lo stato di non raggiungibilità / offline.

---

## Competenze Dimostrate in Questo Laboratorio

* Installazione e configurazione di **Zabbix 7.0** su Ubuntu con database **MariaDB** e frontend **Apache**.
* Configurazione dei parametri di connessione in `/etc/zabbix/zabbix_server.conf` e gestione dei servizi tramite `systemd`.
* Censimento di un host di rete sulla LAN Wi-Fi, individuazione IP tramite `arp-scan` e applicazione del template **ICMP Ping**.
* Personalizzazione della **Dashboard** con widget di disponibilità e grafici di latenza per il monitoraggio immediato.
