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

### Installazione

Repository addon per Home Assistant Supervisor:
```
https://github.com/hexesoft-stefano-rossi/home-assistant-bticino
```

1. Aggiungi il repository in *Impostazioni → Addon → Repository di terze parti*.
2. Installa "Hexesoft BTicino".
3. Configura IP del gateway, password OPEN, broker MQTT e la lista dei dispositivi
   (vedi **DOCS.md** per l'elenco completo dei parametri).
4. Avvia l'addon: i dispositivi compaiono in HA entro pochi secondi.

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

### Installation

Home Assistant Supervisor addon repository:
```
https://github.com/hexesoft-stefano-rossi/home-assistant-bticino
```

1. Add the repository under *Settings → Add-ons → Third-party repositories*.
2. Install "Hexesoft BTicino".
3. Configure gateway IP, OPEN password, MQTT broker and the device list
   (see **DOCS.md** for the full parameter list).
4. Start the addon: devices appear in HA within seconds.

### License

Open-source. See the license file in the repository.
