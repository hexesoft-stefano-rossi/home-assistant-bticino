# Hexesoft Bridge Bticino

Ponte tra l'impianto domotico **BTicino MyHome (bus SCS / OpenWebNet)** e **Home Assistant**, tramite **MQTT**.
Rende disponibili in Home Assistant — in modo automatico e bidirezionale — luci, tapparelle, termostati,
contatori di energia, attuatori, pulsanti e sensori del tuo impianto Bticino.

## Cosa fa

Il bridge si collega al gateway fisico Bticino (es. F454) e:

- **pubblica da solo** tutti i dispositivi che configuri: compaiono in Home Assistant senza scrivere una riga di YAML (MQTT Discovery);
- **ascolta in tempo reale** ogni cambiamento sul bus (un tasto premuto, una tapparella che sale, una temperatura che varia) e aggiorna subito Home Assistant;
- **traduce i comandi** dati da Home Assistant in pacchetti OpenWebNet e li invia sul bus (accendi una luce, imposta un setpoint, apri una tenda);
- **si riconnette da solo** sia al gateway sia al broker MQTT in caso di caduta, e conserva gli stati anche dopo un riavvio.

## Funzionalità

- **Discovery automatica e pulizia**: all'avvio pubblica tutti i dispositivi configurati e **rimuove automaticamente** da Home Assistant quelli che hai tolto dal file di configurazione (niente entità fantasma).
- **Controllo bidirezionale in tempo reale**: ogni evento del bus diventa uno stato MQTT; ogni comando da Home Assistant diventa un frame OpenWebNet.
- **Luci e dimmer**: ON/OFF e regolazione di luminosità (scala 0-255 ↔ 1-10), con supporto ai comandi **temporizzati** (da 0,5 s a 15 min).
- **Tapparelle e tende**: comandi Apri/Chiudi/Stop, stato (in apertura / in chiusura / fermo) e **posizione** (aperta / chiusa / a metà).
- **Termostati diretti**: gestione zona per zona **senza centrale 99 zone**. Caldo/Freddo per singolo termostato, setpoint, temperatura ambiente e **azione** (riscaldamento / raffrescamento / inattivo) per l'icona colorata in Home Assistant. Supporto opzionale alla **ventola fan-coil** (auto / bassa / media / alta).
- **Pompe e valvole**: stato degli attuatori (relè) della termoregolazione, per sapere quando una valvola o la pompa stanno effettivamente lavorando.
- **Monitoraggio energia**: interroga ciclicamente i moduli **F520** e riporta **potenza istantanea (W)** ed **energia totale (kWh)**, già pronta per le statistiche a lungo termine di Home Assistant.
- **Pulsanti**: comandi impulsivi manuali (impulso da 0,5 s).
- **Sensori binari**: contatti, PIR, ausiliari e allarmi, con **inversione logica** (per i contatti normalmente chiusi) e **device class** personalizzabile (icona in HA).
- **Pulsante "Scansiona rete"**: un comando in Home Assistant per rileggere su richiesta lo stato di tutto l'impianto.
- **Diagnostica del gateway**: pubblica modello, indirizzo IP, subnet mask, MAC e versione firmware del web server Bticino.
- **Stato del bridge**: entità Online/Offline (Last Will MQTT), così sai sempre se il ponte è attivo.
- **Log leggibile**: console colorata per livello, con verbosità configurabile (`info`, `debug`, `warning`, `error`), con formato tabellare simmetrico (vedi *Formato del log*).

### Integrazione VRF / gateway (avanzata)
Per le zone configurate come **gateway**, il termostato a muro può pilotare un sistema esterno (es. un VRF Mitsubishi gestito da un bridge dedicato): il bridge inoltra la velocità della ventola scelta in Home Assistant sul canale MQTT condiviso e riaccende/spegne l'icona "in funzione" sul termostato fisico.

## Come funziona (in breve)

Il bridge apre **due connessioni** verso il gateway Bticino (porta 20000):

- una **sessione di ascolto** che intercetta tutto ciò che transita sul bus;
- una **sessione comandi** che invia le richieste e ne attende la conferma (ACK), con riconnessione e keepalive automatici.

Se il gateway richiede la password OPEN, l'autenticazione (HMAC-SHA256) avviene in automatico. Ogni dispositivo è
indicizzato per il proprio topic MQTT, così i messaggi vengono instradati istantaneamente. Gli stati sono pubblicati
con flag *retain*: Home Assistant li ritrova anche dopo un riavvio.

## Formato del log

Ogni riga di log ha una struttura tabellare fissa così da restare **incolonnata** anche in sessioni lunghe:

```
[HH:mm:ss.fff]  LIVELLO | MITT -> DEST | messaggio
```

- **`MITT`** è chi genera l'informazione, **`DEST`** è chi la riceve/consuma.
- Entrambe le sigle sono **sempre di 4 caratteri**, così la freccia `->` e i due `|` cadono nelle stesse colonne per tutta la giornata.

### Sigle usate

| Sigla  | Cosa rappresenta                                                                                          |
|--------|-----------------------------------------------------------------------------------------------------------|
| `CORE` | Sistema/lifecycle: avvio del servizio, arresto pulito, configurazione non valida, host .NET                |
| `HEXE` | Logica interna del Bridge Hexesoft BTicino (Manager, routing driver, cache, indice zone)                   |
| `MQTT` | Broker MQTT: connessione, riconnessione, sottoscrizioni, pubblicazioni grezze                             |
| `SCSB` | Bus SCS BTicino via gateway (F454/F452): frame OpenWebNet inviati, ACK/NACK, eventi in ingresso            |
| `AUTH` | Handshake TCP + autenticazione con il gateway BTicino (fase iniziale della connessione)                    |
| `THRM` | Termostato/dispositivo BTicino a muro (F459 o termostato standalone) — mittente di richieste ventola      |
| `MITS` | Peer Mitsubishi Bridge (canale condiviso `hexesoft_gateway/*`)                                             |
| `HOME` | Home Assistant (Auto-Discovery, pulizia dispositivi fantasma)                                              |

### Esempi

```
[15:32:04.001] INFO   | CORE -> HEXE | Hexesoft Bridge Bticino - Avvio Service
[15:32:04.312] INFO   | AUTH -> HEXE | Sessione attivata (Trusted/No Pwd)
[15:32:04.500] DEBUG  | HEXE -> SCSB | Invio: *#4*31*0##
[15:32:04.612] DEBUG  | SCSB -> HEXE | Ricevuto: ACK
[15:32:04.612] DEBUG  | SCSB -> HEXE | [KNOWN] 2 sala: *#4*31*0*0252##
[15:32:04.613] INFO   | HEXE -> MQTT | 2 sala aggiorna temp/state: 25.2
[15:32:11.404] INFO   | MITS -> HEXE | Bridge Mitsubishi ONLINE: da ora il BT risponderà ai poll #17
[15:32:15.220] INFO   | THRM -> MITS | zona 60: ventola 'alta' -> giro al VRF
[14:29:41.010] INFO   | HEXE -> HOME | Pubblico/Confermo dispositivo: "1 scala nord"
[14:29:44.955] INFO   | HOME -> HEXE | Trovato dispositivo fantasma: homeassistant/climate/.../config
```

Le righe con `[SOPPRESSO]` sono soppressioni volute (frame non instradati, comandi anti-spam identici all'ultimo, eventi retained applicati solo in memoria): compaiono solo in `debug` e aiutano a capire perché una certa azione non è stata eseguita.

## Requisiti

- Un **gateway BTicino** con server OpenWebNet (es. **F454**, MyHOMEServer, web server) raggiungibile in rete sulla **porta 20000**.
- La **password OPEN** del gateway (se ne hai impostata una).
- Un **broker MQTT** (es. Mosquitto) raggiungibile dal bridge e da Home Assistant.
- **Home Assistant** con l'integrazione **MQTT** attiva (la discovery è abilitata di default).

---

## Guida alla Configurazione

Ogni dispositivo va inserito nella categoria corretta del file di configurazione. Due parametri sono comuni alla maggior parte dei dispositivi:

- **`bus`**: identificativo del bus — `0` per il bus locale, `1`-`15` per i bus remoti tramite interfacce F422. *(I termostati usano `zone` al posto di `bus`.)*
- **`model`**: modello hardware (es. `"F411"`, `"H4691"`, `"F520"`), solo a scopo informativo in Home Assistant.

### 1. Luci (`lights`)
Relè ON/OFF e dimmer.
- **`a`**: ambiente (0-9).
- **`pl`**: punto luce (1-15).
- **`timer`** *(opzionale)*: accensione temporizzata. Valori: `none`, `0.5s`, `2s`, `30s`, `1m`, `2m`, `3m`, `4m`, `5m`, `10m`, `15m`.

### 2. Tapparelle e Tende (`shutters`)
Automazioni (WHO 2) con stato e posizione.
- **`a`**: ambiente.
- **`pl`**: punto luce.

### 3. Termostati (`thermostats`)
Termoregolazione (WHO 4), zona per zona.
- **`zone`**: numero della zona termica (1-99).
- **`canHeat`** / **`canCool`**: abilita riscaldamento e/o raffrescamento.
- **`manageFan`**: se `true`, abilita il controllo della ventola fan-coil (`auto`, `low`, `medium`, `high`).

### 4. Contatori Energia (`energy_meters`)
Moduli di misura **F520**.
- **`id`**: indirizzo fisico (1-255). L'indirizzo SCS è calcolato in automatico (es. ID `1` → `51`).

### 5. Interruttori (`switches`)
Attuatori ON/OFF generici.
- **`a`** / **`pl`**: ambiente e punto luce.
- **`timer`** *(opzionale)*: stessi valori delle luci (`0.5s` … `15m`).

### 6. Pulsanti (`buttons`)
Comandi impulsivi manuali.
- **`a`** / **`pl`**: indirizzo fisico.
- Invia un impulso temporizzato di 0,5 s (WHAT 18).

### 7. Relè Termici (`relays`)
Monitoraggio degli attuatori di pompe e valvole della termoregolazione.
- **`bus`**: zona della centrale termica di appartenenza.
- **`id`**: identificativo del relè nella zona.

### 8. Sensori Binari (`binary_sensors`)
Contatti, PIR, ausiliari e allarmi (sola lettura).
- **`a`** / **`pl`**: indirizzo fisico.
- **`invert`**: se `true`, inverte la logica (utile per i contatti normalmente chiusi).
- **`device_class`**: icona/categoria in Home Assistant. Valori: `none`, `door`, `window`, `motion`, `smoke`, `moisture`, `opening`, `presence`, `problem`, `power`.

### Esempio di configurazione

```json
{
  "lights": {
    "devices": [
      { "name": "Sala faretti", "bus": 0, "a": 1, "pl": 1, "model": "F411" },
      { "name": "Scala segnapasso", "bus": 0, "a": 2, "pl": 3, "timer": "5m" }
    ]
  },
  "thermostats": {
    "devices": [
      { "name": "Sala", "zone": 1, "canHeat": true, "canCool": true, "manageFan": false }
    ]
  },
  "energy_meters": {
    "devices": [
      { "name": "Generale", "bus": 0, "id": 1, "model": "F520" }
    ]
  }
}
```