# Anker SOLIX Solarbank 4 — Surplus Solare

[![Importa Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fraw.githubusercontent.com%2Fdinamotechit%2Fha-blueprints%2Fmain%2Fsolarbank4%2Fsurplus_solare.yaml)

Quando la tua Solarbank 4 è carica e il sole produce più di quanto casa consuma, questo blueprint usa l'energia in eccesso per:

- 🔥 **scaldare l'acqua calda sanitaria** — alzando il setpoint della pompa di calore / boiler smart, oppure accendendo una resistenza elettrica via relè
- 🔋 **ricaricare una power station portatile** — via relè o smart plug

E riporta tutto allo stato normale quando il surplus finisce, al tramonto, o se qualcosa va storto (sensori non disponibili, prelievo anomalo dalla rete).

## ✅ Prerequisiti

1. **Home Assistant** aggiornato (2024.6 o successivo)
2. **Integrazione ufficiale [Anker SOLIX Official](https://github.com/anker-charging/ha-anker-solix-official)** installata e connessa alla tua Solarbank 4 via Modbus TCP locale → guarda la nostra guida: [LINK VIDEO]
3. A seconda di cosa vuoi fare col surplus:
   - un sistema ACS integrato in Home Assistant che permetta di regolare la temperatura (es. Daikin Altherma via Onecta), **oppure**
   - un relè smart / smart plug (es. Shelly) collegato alla resistenza o al caricatore → per il collegamento della resistenza guarda la **[Guida Smart Grid](https://youtu.be/hdCO130G4m8)**

## 🚀 Installazione (2 minuti)

1. Clicca il badge **Importa Blueprint** qui sopra
2. Conferma l'indirizzo della tua istanza Home Assistant (solo la prima volta)
3. Clicca **Importa blueprint**
4. Clicca **Crea automazione** e rispondi alle domande del modulo

> In alternativa: *Impostazioni → Automazioni e scene → Blueprint → Importa blueprint* e incolla questo URL:
> ```
> https://raw.githubusercontent.com/dinamotechit/ha-blueprints/main/solarbank4/surplus_solare.yaml
> ```

## ⚙️ Configurazione

Il modulo ti fa **due domande**:

1. **Cosa vuoi fare con l'energia in eccesso?** → power station oppure acqua calda
2. *(solo per l'acqua calda)* **Che tipo di sistema hai?** → temperatura regolabile da Home Assistant, oppure semplice accensione di una resistenza / boiler elettrico

Poi compila **solo la sezione** corrispondente alla tua scelta. I menu dei sensori mostrano solo le entità Anker: scegli quelle della tua Solarbank (SOC, PV Power, Home Load, Max Discharge Power).

### Parametri principali

| Parametro | Default | Note |
|---|---|---|
| Assorbimento del carico | 1500 W | Potenza della PdC in boost ACS, della resistenza o del caricatore della power station. 💡 Molte power station permettono di limitare la potenza di ricarica dall'app: impostala e inserisci qui lo stesso valore |
| SOC di attivazione | 95 % | Il surplus è disponibile sopra questa carica |
| SOC minimo di mantenimento | 50 % | Sotto questa carica il carico viene spento |
| Stabilità attivazione/disattivazione | 5 min | Filtra nuvole e picchi di consumo |
| Prelievo massimo tollerato (facoltativo) | 300 W | Rete di sicurezza: spegne il carico se prelevi dalla rete |

### Il modello fisico (perché non devi impostare soglie a caso)

- **Attivazione**: batteria carica **e** consumo di casa < (potenza massima erogabile − assorbimento del carico). Così quando il carico parte, la Solarbank copre tutto senza prelevare dalla rete.
- **Mantenimento**: il carico resta attivo finché il sole produce almeno l'assorbimento del carico **÷ 1,2** — accetti un piccolo contributo dalla batteria pur di non regalare alla rete il resto del solare.

La potenza massima erogabile viene letta **in tempo reale** dal sensore *Max Discharge Power*: se abbassi il limite di uscita nell'app Anker (es. modalità 800 W), il blueprint se ne accorge da solo e, se il carico non è più alimentabile, ti avvisa con una notifica invece di attivarsi.

## ⚠️ Limitazioni note

- **Power station piena**: il blueprint non rileva quando la power station ha finito di caricarsi; la presa resta alimentata (a vuoto, nessun danno) fino a fine surplus. Una versione futura userà la misura di potenza della smart plug.
- **Modulo non dinamico**: i blueprint di Home Assistant mostrano sempre tutte le sezioni; compila solo quella della modalità scelta, le altre vengono ignorate.
- Il blueprint controlla **un carico**: per più carichi in cascata, crea più automazioni dallo stesso blueprint (con soglie diverse) — o aspetta il blueprint dedicato 😉

## 🔄 Aggiornamenti

Quando pubblichiamo una nuova versione: *Impostazioni → Automazioni e scene → Blueprint → menu ⋮ sul blueprint → **Reimporta***. Le automazioni già create si aggiornano da sole.

## ❤️ Supporta il progetto

Il blueprint è gratuito. Se ti fa risparmiare: [offrici un caffè via PayPal](https://www.paypal.com/ncp/payment/CD3C2UMCK8J7L), visita lo [store](https://dinamotech.it) (codice **DINAMOTECH10**) o iscriviti al [canale](https://www.youtube.com/@DinamoTech).

## 📋 Changelog

- **v1.0** — Prima release: modalità ACS setpoint / ACS resistenza / power station, modello fisico P_max−P_carico, watchdog sensori, blocco con avviso, PV terze parti opzionale, sicurezza sul prelievo rete.
