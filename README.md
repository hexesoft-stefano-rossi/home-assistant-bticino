# Hexesoft Bridge BTicino

Ponte tra un impianto domotico **BTicino MyHome (bus SCS / OpenWebNet)** e **Home Assistant**,
tramite **MQTT**. Rende disponibili in HA — in modo automatico e bidirezionale — luci, tapparelle,
termostati, contatori di energia, attuatori, pulsanti e sensori del tuo impianto Bticino.

---

## 🇮🇹 Italiano

### A cosa serve

Se hai un impianto **BTicino MyHome / SCS** e vuoi integrarlo in **Home Assistant** senza passare
dai cloud Bticino, dall'app NetAtmo o dalla centrale 99 zone, questo bridge fa esattamente quello:
si collega direttamente al tuo **gateway OpenWebNet** (F454, F459, MyHOMEServer, web server, …
porta `20000`) e trasforma ogni dispositivo in un'entità nativa di Home Assistant via MQTT.

Non serve riscrivere YAML di template, non serve mediare con un hub esterno, non serve creare
sensori manualmente: tutto è **MQTT Discovery** standard.

### Cosa fa

- **Auto-discovery completa**: all'avvio pubblica in HA tutti i dispositivi che hai elencato
  nel file di configurazione (luci, tapparelle, termostati, contatori, pulsanti, sensori…),
  e **rimuove** automaticamente quelli che hai tolto — niente entità fantasma.
- **Controllo bidirezionale in tempo reale**: ogni pressione di tasto o variazione sul bus
  arriva in HA nel giro di millisecondi; ogni comando dato da HA diventa un frame OpenWebNet
  sul bus.
- **Si riconnette da solo** sia al gateway BTicino sia al broker MQTT se cade uno dei due,
  e mantiene gli stati (`retain`) anche dopo un riavvio di HA.
- **Autentica** da solo con il gateway (handshake HMAC-SHA256) se hai impostato la password OPEN.

### Dispositivi supportati

| Tipo | Supporto | Note |
|---|---|---|
| **Luci** | ON/OFF, dimmer, temporizzate | Scala HA 0-255 ↔ SCS 1-10, timer da 0,5 s a 15 min |
| **Tapparelle e tende** | Apri/Chiudi/Stop + posizione | Stato "in apertura / in chiusura / fermo" + posizione aperta/chiusa |
| **Termostati** | Setpoint, modalità, azione, ventola fan-coil | Zona per zona, **senza centrale 99 zone**. Caldo/Freddo per singolo termostato, ventola `auto/low/medium/high` opzionale |
| **Contatori energia F520** | 4 sensori pronti per Dashboard Energia | Potenza (W), Energia Totale (kWh), Energia Oggi (kWh), Energia Mese (kWh). `state_class: total_increasing` già impostato |
| **Attuatori / interruttori** | ON/OFF generici, temporizzati | Stessi timer delle luci |
| **Pulsanti** | Impulso momentaneo (0,5 s) | WHAT 18 |
| **Relè pompe e valvole** | Stato degli attuatori di termoregolazione | Per sapere quando pompa/valvola stanno lavorando |
| **Sensori binari** | PIR, contatti, allarmi | **Inversione logica** per contatti NC, `device_class` configurabile |

### Funzionalità avanzate

- **Pulsante "Scansiona rete"**: entità in HA per rileggere su richiesta lo stato di tutto l'impianto.
- **Diagnostica gateway**: modello, IP, netmask, MAC e firmware del gateway pubblicati come sensori.
- **Stato del bridge online/offline** via Last Will MQTT.
- **Push automatico della potenza** (sperimentale, DIM 1200): i contatori F520 inviano la potenza
  solo quando cambia, riducendo il traffico sul bus.
- **Integrazione VRF / gateway**: le zone configurate come gateway inoltrano i comandi ventola a un
  bridge esterno (es. VRF Mitsubishi) tramite canale MQTT condiviso.
- **Log tabellare simmetrico** con sigle a 4 caratteri (`CORE`, `HEXE`, `MQTT`, `SCSB`, `AUTH`,
  `THRM`, `MITS`, `HOME`) per restare incolonnato anche in sessioni lunghe.

### Requisiti

- Un **gateway BTicino OpenWebNet** raggiungibile in rete sulla porta `20000`
  (F454, F459, MyHOMEServer, web server BTicino, ecc.).
- La **password OPEN** del gateway, se impostata.
- Un **broker MQTT** (es. Mosquitto addon di HA).
- **Home Assistant** con integrazione **MQTT** attiva.

### Installazione in Home Assistant (passo-passo)

L'addon non è nello store ufficiale: va aggiunto come **repository di terze parti**. È un'operazione che si fa una volta sola.

**1. Apri la sezione Addon**

- Dal menu laterale di HA clicca **Impostazioni** (icona ingranaggio).
- Clicca **Installa app** (in alcune versioni appare come *Componenti aggiuntivi* o *Add-ons*).

**2. Aggiungi il repository Hexesoft**

- In alto a destra, clicca il menu con i **tre puntini ⋮**.
- Dal menu scegli **Archivi digitali** (o *Repositories*).
- In basso a destra clicca il bottone **AGGIUNGI**.
- Nel campo **URL** incolla il link del repository:
  - Apri in un'altra scheda del browser `https://github.com/hexesoft-stefano-rossi/home-assistant-bticino`
  - Clicca sul bottone verde **Code** in alto a destra
  - Nella tab **HTTPS** copia l'URL mostrato (clicca sull'icona 📋 a destra)
  - Torna in HA e incolla l'URL nel campo
- Clicca **AGGIUNGI**, attendi la conferma, poi **CHIUDI**.

**3. Installa l'addon**

- Nello Store, scorri verso il basso: compare una nuova sezione **"Hexesoft BTicino Addons"**.
- Clicca sulla card **Hexesoft BTicino**.
- Clicca **INSTALLA** e attendi il download (può richiedere qualche minuto la prima volta).

**4. Configura l'addon**

- Appena installato, apri la scheda **Configurazione**.
- Inserisci:
  - **IP e porta del gateway BTicino** (`host`, `port`)
  - **Password OPEN** del gateway (se impostata)
  - **Broker MQTT** (host, porta, utente, password)
  - L'elenco dei **dispositivi** suddiviso per categoria (luci, tapparelle, termostati, contatori, ecc.)
- Clicca **SALVA**.
- Per l'elenco completo dei parametri e un esempio completo, vedi **DOCS.md**.

**5. Avvia l'addon**

- Torna alla scheda **Info**.
- (Consigliato) Attiva **Watchdog** e **Avvio al boot** — così l'addon riparte da solo in caso di riavvio o blocco.
- Clicca **AVVIA**.
- Apri la scheda **Log** e verifica che compaia la riga `Hexesoft Bridge Bticino - Avvio Service` seguita dalla connessione al gateway e al broker MQTT.

**6. Verifica in Home Assistant**

Entro pochi secondi i dispositivi compaiono automaticamente in *Impostazioni → Dispositivi e servizi → MQTT*, raggruppati per nome. Niente da aggiungere a mano.

### Aggiornamenti futuri

Quando esce una nuova versione, HA la mostra nella pagina dell'addon con un bottone **AGGIORNA**. Nessuna riconfigurazione necessaria: parametri e dispositivi restano invariati.

### Licenza

Open-source. Vedi file di licenza nel repository.

---

## 🇬🇧 English

### What it is

A bridge between a **BTicino MyHome (SCS bus / OpenWebNet)** home automation system and
**Home Assistant**, via **MQTT**. It exposes — automatically and bidirectionally — lights,
shutters, thermostats, energy meters, actuators, buttons and sensors of your BTicino system
as native HA entities.

### Why use it

If you own a **BTicino MyHome / SCS** installation and want to integrate it into
**Home Assistant** without going through BTicino cloud, the NetAtmo app or the 99-zone
central unit, this bridge does exactly that: it connects directly to your **OpenWebNet
gateway** (F454, F459, MyHOMEServer, web server — port `20000`) and turns every device
into a native HA entity via MQTT.

No YAML templates to write, no external hub needed, no manual sensor creation:
everything is standard **MQTT Discovery**.

### What it does

- **Full auto-discovery**: on startup it publishes all devices listed in the configuration
  file (lights, shutters, thermostats, meters, buttons, sensors…) and **removes** those
  you delete from the configuration — no ghost entities.
- **Real-time bidirectional control**: every button press or bus event reaches HA within
  milliseconds; every HA command becomes an OpenWebNet frame on the bus.
- **Auto-reconnect** to both the BTicino gateway and the MQTT broker if either drops,
  retaining state (`retain`) across HA restarts.
- **Automatic authentication** with the gateway (HMAC-SHA256 handshake) if an OPEN
  password is set.

### Supported devices

| Type | Support | Notes |
|---|---|---|
| **Lights** | ON/OFF, dimmer, timed | HA scale 0-255 ↔ SCS 1-10, timer from 0.5 s to 15 min |
| **Shutters & blinds** | Open/Close/Stop + position | Opening/closing/idle state + open/closed position |
| **Thermostats** | Setpoint, mode, action, fan-coil | Zone by zone, **no 99-zone central needed**. Heat/Cool per thermostat, optional `auto/low/medium/high` fan |
| **F520 energy meters** | 4 sensors ready for Energy Dashboard | Power (W), Total Energy (kWh), Energy Today (kWh), Energy Month (kWh). `state_class: total_increasing` pre-set |
| **Actuators / switches** | Generic ON/OFF, timed | Same timer values as lights |
| **Buttons** | Momentary pulse (0.5 s) | WHAT 18 |
| **Pump & valve relays** | State of heating-control actuators | Know when pumps/valves are actually working |
| **Binary sensors** | PIR, contacts, alarms | **Logic inversion** for NC contacts, configurable `device_class` |

### Advanced features

- **"Scan network" button**: HA entity to re-read the whole installation state on demand.
- **Gateway diagnostics**: model, IP, netmask, MAC and firmware of the gateway published
  as sensors.
- **Bridge online/offline status** via MQTT Last Will.
- **Power push mode** (experimental, DIM 1200): F520 meters send power only when it
  changes, reducing bus traffic.
- **VRF / gateway integration**: zones configured as gateway forward fan commands to
  an external bridge (e.g. a Mitsubishi VRF bridge) over a shared MQTT channel.
- **Tabular symmetric log** with 4-character tags (`CORE`, `HEXE`, `MQTT`, `SCSB`,
  `AUTH`, `THRM`, `MITS`, `HOME`) so columns stay aligned in long sessions.

### Requirements

- A **BTicino OpenWebNet gateway** reachable on port `20000`
  (F454, F459, MyHOMEServer, BTicino web server, etc.).
- The gateway's **OPEN password**, if set.
- An **MQTT broker** (e.g. HA Mosquitto addon).
- **Home Assistant** with the **MQTT** integration enabled.

### Installation in Home Assistant (step-by-step)

The addon is not in the official store: it must be added as a **third-party repository**. This is a one-time operation.

**1. Open the Add-ons section**

- From the HA sidebar click **Settings** (gear icon).
- Click **Add-ons** (some versions show *Install app* / *Components*).

**2. Add the Hexesoft repository**

- In the top-right corner click the **three-dot menu ⋮**.
- Select **Repositories**.
- In the bottom-right corner click the **ADD** button.
- In the **URL** field paste the repository link:
  - Open in another browser tab `https://github.com/hexesoft-stefano-rossi/home-assistant-bticino`
  - Click the green **Code** button at the top right
  - In the **HTTPS** tab copy the URL shown (click the 📋 icon on the right)
  - Go back to HA and paste the URL in the field
- Click **ADD**, wait for the confirmation, then **CLOSE**.

**3. Install the addon**

- Scroll down in the Store: a new section **"Hexesoft BTicino Addons"** appears.
- Click the **Hexesoft BTicino** card.
- Click **INSTALL** and wait for the download (it may take a few minutes the first time).

**4. Configure the addon**

- Once installed, open the **Configuration** tab.
- Fill in:
  - **BTicino gateway IP and port** (`host`, `port`)
  - **Gateway OPEN password** (if set)
  - **MQTT broker** (host, port, username, password)
  - The **device list** split by category (lights, shutters, thermostats, meters, etc.)
- Click **SAVE**.
- For the full parameter list and a complete example, see **DOCS.md**.

**5. Start the addon**

- Go back to the **Info** tab.
- (Recommended) Enable **Watchdog** and **Start on boot** — so the addon restarts automatically on reboot or crash.
- Click **START**.
- Open the **Log** tab and check for the line `Hexesoft Bridge Bticino - Avvio Service` followed by the connection to the gateway and the MQTT broker.

**6. Verify in Home Assistant**

Within a few seconds devices appear automatically under *Settings → Devices & Services → MQTT*, grouped by name. Nothing to add manually.

### Future updates

When a new version is released, HA shows it on the addon page with an **UPDATE** button. No reconfiguration needed: parameters and device list stay as they are.

### License

Open-source. See the license file in the repository.
