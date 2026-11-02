# Changelog

## 2.5.7

- **Log ulteriormente pulito**: eliminati altri ~410 SOPPRESSO cosmetici all'avvio. Due fix:
  - `BridgeManager`: sostituita la blacklist dei suffissi di stato (`/state`, `/position/state`, `/power`, `/position`) con una **whitelist** che accetta come comando solo i topic che finiscono con `/command`. Prima lasciava passare — e loggava come SOPPRESSO — i retained dei sensori diagnostici `stagione_dedotta`, `tipo_regolazione`, i cleanup `attributes`, e qualunque altro state topic. Ora qualunque echo di stato viene ignorato silenziosamente.
  - `GatewayBridge`: declassati a `trace` i log "frame lucina non inviato (MT non online)" e "frame lucina non inviato (bus non connesso)". Sono race di startup transienti: lo stato viene comunque memorizzato in `_lastLightState` e il primo poll `#17` dopo la connessione lo usa, quindi non c'è alcun impatto funzionale.

## 2.5.6

- **Log più pulito**: eliminati ~180 messaggi `[SOPPRESSO]` cosmetici per avvio tipico. Tre fix mirati:
  - Driver termostato: riconosciuta la DIM 13 (local offset manopola) come no-op silenzioso. Prima generava SOPPRESSO ad ogni polling di stato zona.
  - `BridgeManager`: declassato a `trace` il log di dedup "valore invariato" sui frame SCS → MQTT. Non è un errore, è l'anti-rimbalzo che funziona.
  - `BridgeManager`: ignorati silenziosamente i payload MQTT vuoti nel ramo comando (retained-cleanup del bridge stesso, es. topic legacy `attributes`).
  - `GatewayBridge`: invertito ordine sottoscrizioni (`status` prima di `climate/#`), così il retained `online` di MT arriva prima dei retained light e la decisione "MT online?" all'avvio è presa su uno stato già corretto.

## 2.5.5

- **Fix contatori che diventano "non disponibile"**: i sensori `_energy`, `_energy_today`, `_energy_month` e `_power` ora ripubblicano su MQTT ad ogni polling anche se il valore non è cambiato. Senza questa modifica, su carichi stabili (frigo in standby, meter dormienti) HA marcava il sensore unavailable dopo `expire_after` perché il bridge non rinfrescava `last_updated`.

## 2.5.4

- Aggiunti sensori **Energia Oggi** (DIM 54) e **Energia Mese** (DIM 53) per ogni contatore F520.
- Warm-up iniziale ottimizzato: le gauge di potenza compaiono entro 1-2 minuti dall'avvio invece di 6.
- `expire_after` esteso per i sensori energy per evitare "Non disponibile" durante i cicli di polling lenti.
- Nuova opzione sperimentale `enable_power_push` + `power_push_interval_minutes` per attivare l'auto-aggiornamento potenza dei contatori (default: off).
- Correzioni al parsing dei valori restituiti dai contatori F520 (energia totale e parziale).
- Log standardizzati con prefissi a 4 caratteri (CORE / HEXE / MQTT / SCSB / AUTH / THRM / MITS / HOME).

## 2.5.3

- Rifinitura log e messaggi di avvio.

## 2.3.0

- Versione stabile precedente con funzionalità base: luci, tapparelle, termostati, F459, contatori energia.
