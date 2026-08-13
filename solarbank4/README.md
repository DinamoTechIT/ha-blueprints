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

> 💡 Nell'interfaccia italiana di Home Assistant i blueprint si chiamano **Progetti**.

1. Clicca il badge **Importa Blueprint** qui sopra
2. Conferma l'indirizzo della tua istanza Home Assistant (solo la prima volta)
3. Clicca **Importa progetto**
4. Clicca **Crea automazione** e rispondi alle domande del modulo

> In alternativa: *Impostazioni → Automazioni e scene → Progetti → Importa progetto* e incolla questo URL:
> ```
> https://raw.githubusercontent.com/dinamotechit/ha-blueprints/main/solarbank4/surplus_solare.yaml
> ```

## ⚙️ Configurazione

A vista trovi solo l'essenziale:

1. **I 3 sensori della Solarbank** (SOC, Solar Power, Home Load) — i menu mostrano solo le entità Anker
2. **Cosa vuoi fare con l'energia in eccesso?** → power station oppure acqua calda
3. **Potenza del carico da alimentare** in watt

Poi apri **solo la sezione** della modalità scelta: *Configurazione Acqua Calda* (dove rispondi anche alla domanda sul tipo di sistema: setpoint o resistenza) oppure *Configurazione Power Station*. Le altre sezioni chiuse (*Avanzate*, *Soglie e sicurezza*) hanno già default sensati: aprile solo se vuoi regolare qualcosa.

### Parametri principali

| Parametro | Default | Note |
|---|---|---|
| Potenza del carico da alimentare | 1500 W | Potenza della PdC in boost ACS, della resistenza o del caricatore della power station. 💡 Molte power station permettono di limitare la potenza di ricarica dall'app: impostala e inserisci qui lo stesso valore |
| Potenza massima erogabile | 2500 W | Il limite di scarica della tua Solarbank. Abbassalo se nell'app Anker hai impostato un limite di uscita (es. modalità 800 W) |
| SOC di attivazione | 95 % | Il surplus è disponibile sopra questa carica |
| SOC minimo di mantenimento | 50 % | Sotto questa carica il carico viene spento |
| Stabilità attivazione/disattivazione | 5 min | Filtra nuvole e picchi di consumo |
| Prelievo massimo tollerato (facoltativo) | 300 W | Rete di sicurezza: spegne il carico se prelevi dalla rete |
| Sensore potenza smart plug (facoltativo) | — | Solo modalità power station: se la presa smart misura la potenza, la ricarica completata viene rilevata e la presa spenta |
| Potenza minima di ricarica | 25 W | Sotto questa potenza la power station è considerata carica |
| Durata sotto soglia | 10 min | Tempo di conferma prima dello spegnimento (copre anche la rampa iniziale di ricarica) |

### Il modello fisico (perché non devi impostare soglie a caso)

- **Attivazione**: batteria carica **e** consumo di casa < (potenza massima erogabile − assorbimento del carico). Così quando il carico parte, la Solarbank copre tutto senza prelevare dalla rete.
- **Mantenimento**: il carico resta attivo finché il sole produce almeno l'assorbimento del carico **÷ 1,2** — accetti un piccolo contributo dalla batteria pur di non regalare alla rete il resto del solare.

La potenza massima erogabile si imposta a mano (l'integrazione attuale non espone un sensore dedicato): se abbassi il limite di uscita nell'app Anker, aggiorna anche il valore nel blueprint. Se il carico non è alimentabile con il limite impostato, il blueprint ti avvisa con una notifica invece di attivarsi. Utenti avanzati in **controllo third-party**: nel campo facoltativo potete selezionare l'entità *Target Grid Power* e il limite verrà letto **in tempo reale** dal suo attributo `max_discharge_power`.

## ⚠️ Limitazioni note

- **Power station piena**: se la presa smart non misura la potenza (o il sensore non è configurato), il blueprint non rileva la fine della ricarica e la presa resta alimentata (a vuoto, nessun danno) fino a fine surplus. Con una smart plug con misura di potenza, configura il sensore nella sezione Power Station e la presa si spegne da sola a ricarica completata.
- **Modulo non dinamico**: i blueprint di Home Assistant mostrano sempre tutte le sezioni; compila solo quella della modalità scelta, le altre vengono ignorate.
- Il blueprint controlla **un carico**: per più carichi in cascata, crea più automazioni dallo stesso blueprint (con soglie diverse) — o aspetta il blueprint dedicato 😉

## 🔄 Aggiornamenti

Quando pubblichiamo una nuova versione: *Impostazioni → Automazioni e scene → Progetti → menu ⋮ sul progetto → **Reimporta***. Le automazioni già create si aggiornano da sole.

## ❤️ Supporta il progetto

Il blueprint è gratuito. Se ti fa risparmiare: [offrici un caffè via PayPal](https://www.paypal.com/ncp/payment/CD3C2UMCK8J7L), visita lo [store](https://dinamotech.it) (codice **DINAMOTECH10**) o iscriviti al [canale](https://www.youtube.com/@DinamoTech).

## 📋 Changelog

- **v1.2.1** — Testi del form più chiari: "Assorbimento del carico" diventa "Potenza del carico da alimentare" con i tre casi a elenco; i tag [Setpoint]/[Resistenza] compaiono anche nelle opzioni della domanda sul tipo di sistema e ogni campo ACS dichiara per quale scelta va compilato; descrizioni aggiunte a soglie SOC e tempi di stabilità; "Solo con il sole" diventa "Spegni al tramonto" con spiegazione. Nessuna modifica alla logica.
- **v1.2** — Form semplificato: a vista restano solo i 3 sensori, la scelta del surplus e l'assorbimento del carico; potenza massima, lettura live third-party e PV di terze parti sono nella nuova sezione chiusa *Avanzate*; *Soglie e comportamento* e *Sicurezza* sono ora un'unica sezione *Soglie e sicurezza*. Descrizione introduttiva accorciata. Solo riorganizzazione della presentazione: nessun input rimosso o rinominato, le automazioni esistenti non cambiano.
- **v1.1.1** — Potenza massima erogabile: l'integrazione non espone il sensore "Max Discharge Power" (è un valore interno), quindi il valore manuale diventa l'impostazione principale. Il campo entità resta come opzione avanzata: in controllo third-party si può selezionare *Target Grid Power* e il limite viene letto live dal suo attributo `max_discharge_power`. Nessuna modifica necessaria alle automazioni esistenti.
- **v1.1** — Rilevamento power station carica (opzionale): nuovo sensore di potenza della smart plug nella sezione Power Station; se la potenza resta sotto la soglia (default 25 W) per il tempo impostato (default 10 min) mentre la presa è accesa, la presa viene spenta. Le automazioni esistenti continuano a funzionare senza modifiche: basta Reimportare il blueprint.
- **v1.0** — Prima release: modalità ACS setpoint / ACS resistenza / power station, modello fisico P_max−P_carico, watchdog sensori, blocco con avviso, PV terze parti opzionale, sicurezza sul prelievo rete.
