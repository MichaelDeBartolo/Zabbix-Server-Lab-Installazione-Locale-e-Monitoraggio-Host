## 📌 Architettura del Lab

Quando si installa Zabbix in locale, l'intera infrastruttura si fonda sulle risorse hardware della macchina ospitante:
* **Storage (Disco):** Tutte le configurazioni, i dati storici, i log e i grafici vengono memorizzati fisicamente sul disco rigido (SSD/HDD) tramite il database relazionale (MariaDB/MySQL).
* **Calcolo (RAM/CPU):** I demoni di Zabbix Server, lo Zabbix Agent e il web server Apache utilizzano la memoria RAM e i core della CPU locale per elaborare ed esporre le metriche in tempo reale.

---

## 🛠️ Fase 1: Configurazione Ottimale del Database

Per consentire a Zabbix Server di comunicare correttamente con il database locale, il file di configurazione `/etc/zabbix/zabbix_server.conf` è stato impostato attivando la direttiva dedicata alla password, mantenendo i commenti descrittivi isolati:

```ini
### Option: DBPassword
#       Database password.
#       Comment this line if no password is used.

DBPassword=passwordtest
Una volta salvato il file, il servizio è stato avviato e verificato con i seguenti comandi di sistema:

Bash
sudo systemctl restart zabbix-server
sudo systemctl status zabbix-server
🌐 Fase 2: Accesso al Frontend Web
L'interfaccia di gestione è raggiungibile localmente da browser all'indirizzo: http://localhost/zabbix.

Le credenziali predefinite per il primo login amministrativo sono:

Username: Admin (con la 'A' maiuscola)

Password: zabbix

📱 Fase 3: Configurazione e Monitoraggio Host (Samsung S24)
Per testare le capacità di monitoraggio di rete senza installare agenti software di terze parti, è stato censito uno smartphone come host.

1. Identificazione dell'IP Locale dello Smartphone
Assicurandosi che lo smartphone fosse connesso alla stessa rete Wi-Fi del computer, l'indirizzo IP locale (IPv4) è stato ricavato dalle proprietà avanzate della rete sul telefono.

Nota: In alternativa è possibile scansionare i dispositivi vivi nella sottorete dal terminale del server usando:

Bash
sudo arp-scan --localnet
2. Creazione dell'Host in Zabbix 7.0
All'interno del pannello web di Zabbix, la procedura guidata ha previsto i seguenti inserimenti sotto Data collection ➡️ Hosts ➡️ Create host:

Host name: Samsung s24

Host groups: Discovered hosts (Gruppo obbligatorio per il salvataggio del record).

Interfaces: È stata aggiunta un'interfaccia di tipo Agent valorizzando il campo IP address con l'IP statico/corrente dello smartphone e lasciando la spunta su IP.

3. Logica del Template (ICMP Ping)
Per istruire Zabbix su quali dati raccogliere, all'host è stato associato il template ICMP Ping (selezionabile manualmente dal gruppo Templates/Modules).

Questo modello permette a Zabbix di testare la presenza del telefono inviando pacchetti di rete regolari, generando metriche per:

Stato dell'host (Dispositivo Up/Down)

Packet Loss (Percentuale di pacchetti persi nel Wi-Fi)

Response Time (Tempo di risposta/latenza in millisecondi)

📊 Visualizzazione dei Dati e Dashboard Principale
Una volta completata la configurazione, i dati e i grafici sono stati esposti in due sezioni principali:

Latest Data: Raggiungibile da Monitoring ➡️ Latest data selezionando l'host Samsung s24 per osservare l'aggiornamento numerico dei record in tempo reale.

Global Dashboard: Per una visualizzazione immediata, la Dashboard principale è stata personalizzata in modalità Edit inserendo due Widget dedicati:

Host availability: Un riquadro visivo che mostra lo stato di raggiungibilità del gruppo di host.

Graph (classic): Un grafico lineare dinamico impostato su Simple graph che punta direttamente alla metrica ICMP response time dell'host creato.

Grazie a questa configurazione, se lo smartphone si disconnette dalla rete Wi-Fi, il grafico mostra immediatamente l'interruzione della linea e la dashboard segnala lo stato di Offline.
