# Hexesoft Bridge BTicino — Documentazione / Documentation

Manuale di configurazione dell'addon. Per una panoramica sulle funzionalità vedi **README.md**.

Configuration manual for the addon. For a feature overview see **README.md**.

---

## 🇮🇹 Italiano

### 1. Prerequisiti

Prima di avviare l'addon verifica di avere:

- **Gateway BTicino OpenWebNet** (F454, F459, MyHOMEServer, web server) raggiungibile sulla rete locale sulla porta `20000`.
- **IP statico** del gateway (consigliato: fissalo nel DHCP del router).
- **Password OPEN** del gateway (se impostata) — la trovi nella configurazione del gateway stesso.
- **Broker MQTT** già configurato in HA (es. addon ufficiale Mosquitto).
- Credenziali di accesso al broker MQTT (username + password).
- **Home Assistant** con integrazione **MQTT** attiva (Discovery abilitata — è il default).

### 2. Installazione

1. Vai in *Impostazioni → Addon → Repository di terze parti*.
2. Aggiungi `https://github.com/hexesoft-stefano-rossi/home-assistant-bticino`.
3. Installa **Hexesoft BTicino**.
4. Apri la scheda *Configurazione* dell'addon e compila i parametri descritti sotto.
5. Avvia l'addon e controlla la scheda *Log*.

### 3. Parametri di configurazione

Tutta la configurazione sta nel file `config.yaml` (gestito dall'interfaccia Supervisor di HA).

#### 3.1 Sezione `bticino`

Connessione al gateway.

| Parametro | Tipo | Obbligatorio | Descrizione |
|---|---|---|---|
| `host` | stringa | ✅ | IP del gateway (es. `192.168.1.50`) |
| `port` | intero | ✅ | Porta OpenWebNet (default `20000`) |
| `password` | stringa | se gateway protetto | Password OPEN. Vuota se il gateway è aperto |

#### 3.2 Sezione `mqtt`

Connessione al broker.

| Parametro | Tipo | Obbligatorio | Descrizione |
|---|---|---|---|
| `host` | stringa | ✅ | Host del broker (default `core-mosquitto` per l'addon ufficiale) |
| `port` | intero | ✅ | Porta broker (default `1883`) |
| `username` | stringa | ⚠️ | Utente MQTT |
| `password` | stringa | ⚠️ | Password MQTT |
| `base_topic` | stringa | ✅ | Prefisso topic MQTT del bridge (default `bticino_bridge`) |
| `discovery_prefix` | stringa | ✅ | Prefisso Discovery di HA (default `homeassistant`) |

#### 3.3 Sezione `scan_settings`

Regole di scansione e polling.

| Parametro | Tipo | Default | Descrizione |
|---|---|---|---|
| `skiplight` | bool | `false` | Non scansionare le luci all'avvio |
| `skipshutter` | bool | `false` | Non scansionare le tapparelle all'avvio |
| `skipthermostat` | bool | `false` | Non scansionare i termostati all'avvio |
| `disableenergymonitor` | bool | `false` | Disattiva completamente il monitor energia |
| `energypollingintervalseconds` | 30-3600 | `60` | Intervallo polling dei contatori energia (secondi) |
| `enable_power_push` | bool | `false` | Attiva il push spontaneo della potenza (DIM 1200) sui contatori F520 |
| `power_push_interval_minutes` | 1-255 | `5` | Minuti minimi fra due frame potenza spontanei |

> ⚠️ **`enable_power_push`** è **sperimentale**: non tutti i firmware F520 lo supportano.
> Dopo l'attivazione controlla il log: se vedi `DIM 1200 NACK` o `Nessun contatore ha accettato il push potenza`, riporta a `false` e il polling normale riprende.

#### 3.4 Sezione `system_settings`

| Parametro | Valori ammessi | Default | Descrizione |
|---|---|---|---|
| `log_level` | `info`, `debug`, `warning`, `error` | `info` | Verbosità del log |

### 4. Categorie di dispositivi

Due parametri sono comuni alla maggior parte dei dispositivi:

- **`name`**: nome mostrato in HA (obbligatorio).
- **`bus`**: `0` per il bus locale; `1-15` per i bus remoti via interfacce F422.
- **`model`** *(opzionale)*: modello hardware mostrato in HA solo a scopo informativo (es. `"F411"`).

> I termostati usano `zone` al posto di `bus`.

#### 4.1 Luci — `lights`

Relè ON/OFF e dimmer.

| Campo | Tipo | Descrizione |
|---|---|---|
| `a` | 0-15 | Ambiente |
| `pl` | 0-15 | Punto luce |
| `timer` | enum | Accensione temporizzata, opzionale: `none`, `0.5s`, `2s`, `30s`, `1m`, `2m`, `3m`, `4m`, `5m`, `10m`, `15m` |

#### 4.2 Tapparelle e tende — `shutters`

| Campo | Tipo | Descrizione |
|---|---|---|
| `a` | 0-15 | Ambiente |
| `pl` | 0-15 | Punto luce |

#### 4.3 Termostati — `thermostats`

| Campo | Tipo | Descrizione |
|---|---|---|
| `zone` | 0-99 | Numero della zona termica |
| `canHeat` | bool | Abilita riscaldamento |
| `canCool` | bool | Abilita raffrescamento |
| `manageFan` | bool | Abilita il controllo della ventola fan-coil (`auto`, `low`, `medium`, `high`) |
| `startup_preset` | `comfort`/`economy` | Preset applicato all'avvio |

#### 4.4 Contatori energia — `energy_meters`

Moduli di misura **F520**.

| Campo | Tipo | Descrizione |
|---|---|---|
| `id` | intero | Indirizzo fisico del contatore (1-255). L'indirizzo SCS è calcolato in automatico (es. ID `1` → `51`) |

**Entità generate automaticamente per ciascun contatore:**

| Entità | Unità | DIM | Note |
|---|---|---|---|
| Potenza istantanea | W | 113 | Pollata ciclicamente (o ricevuta in push se abilitato) |
| Energia Totale | kWh | 51 | Totalizzatore cumulativo |
| Energia Oggi | kWh | 54 | Si azzera a mezzanotte |
| Energia Mese | kWh | 53 | Si azzera a fine mese |

I 3 sensori di energia hanno `state_class: total_increasing`: puoi usare direttamente **Energia Oggi** come fonte giornaliera nel Dashboard Energia di HA, senza template.

#### 4.5 Interruttori — `switches`

Attuatori ON/OFF generici.

| Campo | Tipo | Descrizione |
|---|---|---|
| `a` | 0-15 | Ambiente |
| `pl` | 0-15 | Punto luce |
| `timer` | enum | Stessi valori delle luci |

#### 4.6 Pulsanti — `buttons`

Comandi impulsivi manuali, impulso fisso 0,5 s (WHAT 18).

| Campo | Tipo | Descrizione |
|---|---|---|
| `a` | 0-15 | Ambiente |
| `pl` | 0-15 | Punto luce |

#### 4.7 Relè termici — `relays`

Monitoraggio degli attuatori di pompe e valvole della termoregolazione.

| Campo | Tipo | Descrizione |
|---|---|---|
| `bus` | 0-15 | Zona della centrale termica di appartenenza |
| `id` | intero | Identificativo del relè nella zona |

#### 4.8 Sensori binari — `binary_sensors`

Contatti, PIR, ausiliari, allarmi (sola lettura).

| Campo | Tipo | Descrizione |
|---|---|---|
| `a` | 0-15 | Ambiente |
| `pl` | 0-15 | Punto luce |
| `invert` | bool | Inverte la logica (utile per contatti normalmente chiusi) |
| `device_class` | enum | Icona/categoria in HA: `none`, `door`, `window`, `motion`, `smoke`, `moisture`, `opening`, `presence`, `problem`, `power` |

### 5. Esempio completo di configurazione

```yaml
bticino:
  host: "192.168.1.50"
  port: 20000
  password: "12345"

mqtt:
  host: "core-mosquitto"
  port: 1883
  username: "mqttuser"
  password: "mqttpass"
  base_topic: "bticino_bridge"
  discovery_prefix: "homeassistant"

scan_settings:
  skiplight: false
  skipshutter: false
  skipthermostat: false
  disableenergymonitor: false
  energypollingintervalseconds: 60

system_settings:
  log_level: "info"

lights:
  devices:
    - name: "Sala faretti"
      bus: 0
      a: 1
      pl: 1
      model: "F411"
    - name: "Scala segnapasso"
      bus: 0
      a: 2
      pl: 3
      timer: "5m"

shutters:
  devices:
    - name: "Sala tenda"
      bus: 0
      a: 1
      pl: 5

thermostats:
  devices:
    - name: "Sala"
      zone: 1
      canHeat: true
      canCool: true
      manageFan: false

energy_meters:
  devices:
    - name: "Generale"
      bus: 0
      id: 1
      model: "F520"

binary_sensors:
  devices:
    - name: "PIR ingresso"
      bus: 0
      a: 1
      pl: 1
      invert: false
      device_class: motion
```

### 6. Il log

Ogni riga ha una struttura tabellare fissa, incolonnata:

```
[HH:mm:ss.fff]  LIVELLO | MITT -> DEST | messaggio
```

- **`MITT`** = chi genera l'informazione, **`DEST`** = chi la riceve/consuma.
- Entrambe le sigle sono **sempre di 4 caratteri**.

#### Sigle usate

| Sigla | Significato |
|---|---|
| `CORE` | Sistema/lifecycle: avvio, arresto, configurazione, host .NET |
| `HEXE` | Logica interna del Bridge (Manager, routing, cache, indice zone) |
| `MQTT` | Broker MQTT: connessione, sottoscrizioni, pubblicazioni |
| `SCSB` | Bus SCS: frame OpenWebNet inviati, ACK/NACK, eventi in ingresso |
| `AUTH` | Handshake e autenticazione col gateway |
| `THRM` | Termostato/dispositivo BTicino a muro |
| `MITS` | Peer Mitsubishi Bridge (canale condiviso `hexesoft_gateway/*`) |
| `HOME` | Home Assistant (Discovery, pulizia dispositivi fantasma) |

Le righe `[SOPPRESSO]` sono volute: frame non instradati, anti-rimbalzo, eventi retained applicati solo in memoria. Compaiono in `debug` per capire perché una certa azione non è stata eseguita — non sono errori.

### 7. Troubleshooting

| Problema | Causa probabile | Verifica |
|---|---|---|
| L'addon parte, poi si ferma | Config mancante o errata | Controlla il log: `Manca l'indirizzo IP del Gateway` o `Manca l'indirizzo IP del broker MQTT` |
| Dispositivi non compaiono in HA | MQTT Discovery disattivata | *Impostazioni → Integrazioni → MQTT → Configura → Discovery on* |
| `AUTH -> HEXE | Password errata` | Password OPEN sbagliata | Controlla la password nella config del gateway |
| `Contatori NACK su DIM 1200` | Firmware F520 vecchio | Imposta `enable_power_push: false` |
| Entità "fantasma" dopo rimozione | Retained MQTT non pulito | Il bridge le rimuove da solo al riavvio; se non basta, pulisci il topic da MQTT Explorer |
| Lo stato del termostato non si aggiorna | `canHeat`/`canCool` non abilitati | Attiva quelli che usa l'impianto |

---

## 🇬🇧 English

### 1. Prerequisites

Before starting the addon, make sure you have:

- A **BTicino OpenWebNet gateway** (F454, F459, MyHOMEServer, web server) reachable on the local network on port `20000`.
- A **static IP** for the gateway (recommended: pin it in your router DHCP).
- The **OPEN password** of the gateway (if set) — find it in the gateway's own configuration.
- An **MQTT broker** already configured in HA (e.g. the official Mosquitto addon).
- MQTT broker credentials (username + password).
- **Home Assistant** with the **MQTT** integration enabled (Discovery on — default).

### 2. Installation

1. Go to *Settings → Add-ons → Third-party repositories*.
2. Add `https://github.com/hexesoft-stefano-rossi/home-assistant-bticino`.
3. Install **Hexesoft BTicino**.
4. Open the addon *Configuration* tab and fill in the parameters below.
5. Start the addon and watch the *Log* tab.

### 3. Configuration parameters

The whole configuration lives in `config.yaml` (edited through the Supervisor UI).

#### 3.1 `bticino` section

Gateway connection.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `host` | string | ✅ | Gateway IP (e.g. `192.168.1.50`) |
| `port` | int | ✅ | OpenWebNet port (default `20000`) |
| `password` | string | if gateway is protected | OPEN password. Empty if gateway is open |

#### 3.2 `mqtt` section

Broker connection.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `host` | string | ✅ | Broker host (default `core-mosquitto` for the official addon) |
| `port` | int | ✅ | Broker port (default `1883`) |
| `username` | string | ⚠️ | MQTT username |
| `password` | string | ⚠️ | MQTT password |
| `base_topic` | string | ✅ | Bridge MQTT topic prefix (default `bticino_bridge`) |
| `discovery_prefix` | string | ✅ | HA Discovery prefix (default `homeassistant`) |

#### 3.3 `scan_settings` section

Scan and polling rules.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `skiplight` | bool | `false` | Skip lights scan at startup |
| `skipshutter` | bool | `false` | Skip shutters scan at startup |
| `skipthermostat` | bool | `false` | Skip thermostats scan at startup |
| `disableenergymonitor` | bool | `false` | Fully disable the energy monitor |
| `energypollingintervalseconds` | 30-3600 | `60` | Energy meter polling interval (seconds) |
| `enable_power_push` | bool | `false` | Enable spontaneous power push (DIM 1200) on F520 meters |
| `power_push_interval_minutes` | 1-255 | `5` | Minimum minutes between two spontaneous power frames |

> ⚠️ **`enable_power_push`** is **experimental**: not all F520 firmware revisions support it.
> After enabling, watch the log: if you see `DIM 1200 NACK` or `No meter accepted the power push`, revert to `false` and normal polling resumes.

#### 3.4 `system_settings` section

| Parameter | Allowed values | Default | Description |
|---|---|---|---|
| `log_level` | `info`, `debug`, `warning`, `error` | `info` | Log verbosity |

### 4. Device categories

Two fields are common to most devices:

- **`name`**: label shown in HA (mandatory).
- **`bus`**: `0` for the local bus; `1-15` for remote buses via F422 interfaces.
- **`model`** *(optional)*: hardware model shown in HA for reference (e.g. `"F411"`).

> Thermostats use `zone` instead of `bus`.

#### 4.1 Lights — `lights`

ON/OFF relays and dimmers.

| Field | Type | Description |
|---|---|---|
| `a` | 0-15 | Environment |
| `pl` | 0-15 | Light point |
| `timer` | enum | Timed switch-on, optional: `none`, `0.5s`, `2s`, `30s`, `1m`, `2m`, `3m`, `4m`, `5m`, `10m`, `15m` |

#### 4.2 Shutters & blinds — `shutters`

| Field | Type | Description |
|---|---|---|
| `a` | 0-15 | Environment |
| `pl` | 0-15 | Light point |

#### 4.3 Thermostats — `thermostats`

| Field | Type | Description |
|---|---|---|
| `zone` | 0-99 | Thermal zone number |
| `canHeat` | bool | Enable heating |
| `canCool` | bool | Enable cooling |
| `manageFan` | bool | Enable fan-coil control (`auto`, `low`, `medium`, `high`) |
| `startup_preset` | `comfort`/`economy` | Preset applied at startup |

#### 4.4 Energy meters — `energy_meters`

**F520** metering modules.

| Field | Type | Description |
|---|---|---|
| `id` | int | Physical meter address (1-255). SCS address is computed automatically (e.g. ID `1` → `51`) |

**Entities created automatically for each meter:**

| Entity | Unit | DIM | Notes |
|---|---|---|---|
| Instant power | W | 113 | Polled cyclically (or received as push if enabled) |
| Total Energy | kWh | 51 | Cumulative totalizer |
| Energy Today | kWh | 54 | Resets at midnight |
| Energy Month | kWh | 53 | Resets at end of month |

All 3 energy sensors have `state_class: total_increasing`: you can use **Energy Today** directly as a daily source in HA's Energy Dashboard, no templates required.

#### 4.5 Switches — `switches`

Generic ON/OFF actuators.

| Field | Type | Description |
|---|---|---|
| `a` | 0-15 | Environment |
| `pl` | 0-15 | Light point |
| `timer` | enum | Same values as lights |

#### 4.6 Buttons — `buttons`

Momentary pulse commands, fixed 0.5 s pulse (WHAT 18).

| Field | Type | Description |
|---|---|---|
| `a` | 0-15 | Environment |
| `pl` | 0-15 | Light point |

#### 4.7 Heating relays — `relays`

State monitoring of pump and valve actuators of the heating control system.

| Field | Type | Description |
|---|---|---|
| `bus` | 0-15 | Thermal zone the relay belongs to |
| `id` | int | Relay identifier within the zone |

#### 4.8 Binary sensors — `binary_sensors`

Contacts, PIRs, auxiliaries, alarms (read-only).

| Field | Type | Description |
|---|---|---|
| `a` | 0-15 | Environment |
| `pl` | 0-15 | Light point |
| `invert` | bool | Invert logic (useful for normally-closed contacts) |
| `device_class` | enum | Icon/category in HA: `none`, `door`, `window`, `motion`, `smoke`, `moisture`, `opening`, `presence`, `problem`, `power` |

### 5. Full configuration example

```yaml
bticino:
  host: "192.168.1.50"
  port: 20000
  password: "12345"

mqtt:
  host: "core-mosquitto"
  port: 1883
  username: "mqttuser"
  password: "mqttpass"
  base_topic: "bticino_bridge"
  discovery_prefix: "homeassistant"

scan_settings:
  skiplight: false
  skipshutter: false
  skipthermostat: false
  disableenergymonitor: false
  energypollingintervalseconds: 60

system_settings:
  log_level: "info"

lights:
  devices:
    - name: "Living room spots"
      bus: 0
      a: 1
      pl: 1
      model: "F411"
    - name: "Staircase step-light"
      bus: 0
      a: 2
      pl: 3
      timer: "5m"

shutters:
  devices:
    - name: "Living room curtain"
      bus: 0
      a: 1
      pl: 5

thermostats:
  devices:
    - name: "Living room"
      zone: 1
      canHeat: true
      canCool: true
      manageFan: false

energy_meters:
  devices:
    - name: "Main"
      bus: 0
      id: 1
      model: "F520"

binary_sensors:
  devices:
    - name: "Entrance PIR"
      bus: 0
      a: 1
      pl: 1
      invert: false
      device_class: motion
```

### 6. The log

Every line has a fixed tabular structure:

```
[HH:mm:ss.fff]  LEVEL | SRC  -> DEST | message
```

- **`SRC`** = producer of the information, **`DEST`** = consumer.
- Both tags are **always 4 characters**.

#### Tags used

| Tag | Meaning |
|---|---|
| `CORE` | System/lifecycle: startup, shutdown, configuration, .NET host |
| `HEXE` | Internal bridge logic (Manager, routing, cache, zone index) |
| `MQTT` | MQTT broker: connection, subscriptions, publications |
| `SCSB` | SCS bus: OpenWebNet frames sent, ACK/NACK, incoming events |
| `AUTH` | Handshake and authentication with the gateway |
| `THRM` | BTicino wall thermostat/device |
| `MITS` | Mitsubishi Bridge peer (shared `hexesoft_gateway/*` channel) |
| `HOME` | Home Assistant (Discovery, ghost-device cleanup) |

Lines marked `[SOPPRESSO]` (`suppressed`) are intentional: unrouted frames, anti-bounce dedup, retained events applied only in memory. They show only at `debug` level and help understanding why a given action was not performed — they are not errors.

### 7. Troubleshooting

| Problem | Likely cause | What to check |
|---|---|---|
| Addon starts then stops | Missing or wrong config | Check the log: `Missing Gateway IP` or `Missing MQTT broker IP` |
| Devices don't appear in HA | MQTT Discovery disabled | *Settings → Integrations → MQTT → Configure → Discovery on* |
| `AUTH -> HEXE | Wrong password` | Wrong OPEN password | Check the password in the gateway configuration |
| `Meters NACK on DIM 1200` | Old F520 firmware | Set `enable_power_push: false` |
| "Ghost" entities after removal | Retained MQTT not cleaned | The bridge cleans them at restart; if still there, purge the topic with MQTT Explorer |
| Thermostat state doesn't update | `canHeat`/`canCool` not enabled | Enable the ones your installation uses |
