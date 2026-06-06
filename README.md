## 📌 Architettura del Lab

Quando si installa Zabbix in locale, l'intera infrastruttura si fonda sulle risorse hardware della macchina ospitante:
* **Storage (Disco):** Tutte le configurazioni, i dati storici, i log e i grafici vengono memorizzati fisicamente sul disco rigido tramite il database relazionale MariaDB/MySQL.
* **Calcolo (RAM/CPU):** Il server, l'agente locale e il web server Apache utilizzano la memoria e i core della CPU del PC per elaborare ed esporre le metriche in tempo reale.

---

## 🛠️ FASE 1: Installazione dei Pacchetti Zabbix

Per installare Zabbix Server, il frontend web e l'agente locale sul sistema operativo Ubuntu, sono stati eseguiti i seguenti comandi da terminale:

```bash
# 1. Scaricamento e installazione del repository ufficiale Zabbix 7.0
wget [https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_7.0-1+ubuntu24.04_all.deb](https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_7.0-1+ubuntu24.04_all.deb)
sudo dpkg -i zabbix-release_7.0-1+ubuntu24.04_all.deb
sudo apt update

# 2. Installazione dei componenti core di Zabbix e del web server Apache
sudo apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-agent -y

```

## ⚙️ FASE 2: Configurazione del Database e del Server

Dopo aver creato il database su MariaDB e importato lo schema iniziale, è stato modificato il file di configurazione principale di Zabbix /etc/zabbix/zabbix_server.conf per inserire le credenziali di accesso:

```bash

Ini, TOML
### Option: DBPassword
#       Database password.
#       Comment this line if no password is used.

DBPassword=passwordtest
Una volta salvato il file con la password corretta, i servizi sono stati avviati e abilitati per partire in automatico al boot del PC:

Bash
sudo systemctl restart zabbix-server zabbix-agent apache2
sudo systemctl enable zabbix-server zabbix-agent apache2

```

## 🌐 FASE 3: Accesso al Frontend Web
L'interfaccia grafica di gestione risponde localmente sul browser all'indirizzo: http://localhost/zabbix.

I parametri di login predefiniti per il primo accesso amministrativo sono:

Username: Admin (con la A maiuscola)

Password: zabbix

## 📱 FASE 4: Configurazione Host (Samsung S24)
Per testare la rete senza installare agenti sul telefono, lo smartphone è stato configurato come host monitorato sfruttando la rete Wi-Fi locale.

Se necessario, l'identificazione dei dispositivi attivi nella stessa sottorete può essere effettuata da terminale tramite:

```bash

Bash
sudo apt install arp-scan -y
sudo arp-scan --localnet

```

All'interno del pannello di Zabbix, sotto Data collection ➡️ Hosts ➡️ Create host, sono stati mappati i seguenti dati:

Host name: Samsung s24

Host groups: Discovered hosts (Gruppo obbligatorio per il salvataggio)

Interfaces: È stata aggiunta un'interfaccia di tipo Agent, inserendo l'indirizzo IP del telefono e lasciando la selezione attiva su IP.

Template: È stato associato il modello ICMP Ping (disponibile nel gruppo Templates/Modules).

Questo template invia pacchetti di rete regolari per misurare:

Raggiungibilità del dispositivo (Online/Offline)

Latenza di risposta Wi-Fi (in millisecondi)

Percentuale di pacchetti persi

## 📊 FASE 5: Visualizzazione Dati e Personalizzazione Dashboard
Per una visualizzazione immediata dello stato dello smartphone senza navigare nei menu interni, la Dashboard principale (Global view) è stata modificata inserendo due elementi visivi dedicati:

Widget Host Availability: Mostra un riquadro colorato che indica se il gruppo di dispositivi è attivo o irraggiungibile.

Widget Graph (classic): Un grafico lineare impostato come Simple graph che punta alla metrica ICMP response time dell'host Samsung s24.

## Grazie a questa impostazione, se lo smartphone si disconnette dal Wi-Fi, il grafico si interrompe immediatamente e la dashboard segnala lo stato di Offline.
