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
| Controllo della ricarica (power station) | smart plug | Smart plug / relè smart comandati dal blueprint. L'opzione "presa di backup della Solarbank" è predisposta ma non ancora disponibile |
| Sensore della potenza assorbita (facoltativo) | — | Solo modalità power station: se la presa smart o il relè misurano la potenza, la ricarica completata viene rilevata e la presa spenta |
| Potenza minima di ricarica | 25 W | Sotto questa potenza la power station è considerata carica |
| Durata sotto soglia | 10 min | Tempo di conferma prima dello spegnimento (copre anche la rampa iniziale di ricarica) |

### Il modello fisico (perché non devi impostare soglie a caso)

- **Attivazione**: batteria carica **e** consumo di casa < (potenza massima erogabile − assorbimento del carico). Così quando il carico parte, la Solarbank copre tutto senza prelevare dalla rete.
- **Mantenimento**: il carico resta attivo finché il sole produce almeno l'assorbimento del carico **÷ 1,2** — accetti un piccolo contributo dalla batteria pur di non regalare alla rete il resto del solare.

Le condizioni di spegnimento (fine surplus, prelievo dalla rete) sono valutate **solo mentre il carico è attivo**: ogni attivazione ri-arma i controlli. Il dettaglio conta con l'**immissione zero** (configurazione tipica in Italia): a batteria piena la Solarbank strozza la produzione sulla domanda di casa, quindi il sensore solare legge valori bassi anche in pieno sole — senza questo accorgimento lo spegnimento potrebbe non scattare quando serve.

La potenza massima erogabile si imposta a mano (l'integrazione attuale non espone un sensore dedicato): se abbassi il limite di uscita nell'app Anker, aggiorna anche il valore nel blueprint. Se il carico non è alimentabile con il limite impostato, il blueprint ti avvisa con una notifica invece di attivarsi.

## ⚠️ Limitazioni note

- **Power station piena**: se la presa o il relè non misurano la potenza (o il sensore non è configurato), il blueprint non rileva la fine della ricarica e la presa resta alimentata (a vuoto, nessun danno) fino a fine surplus. Con una presa smart o un relè dotati di misura di potenza, configura il sensore nella sezione Power Station e la presa si spegne da sola a ricarica completata.
- **Presa di backup della Solarbank**: l'opzione compare nel modulo ma non è ancora attiva — l'integrazione ufficiale non consente ancora di comandarla da Home Assistant. Selezionarla oggi produce solo una notifica di avviso; sarà abilitata in una versione futura e richiederà la Solarbank in modalità **Controllo di terze parti**.
- **Modulo non dinamico**: i blueprint di Home Assistant mostrano sempre tutte le sezioni; compila solo quella della modalità scelta, le altre vengono ignorate.
- Il blueprint controlla **un carico**: per più carichi in cascata, crea più automazioni dallo stesso blueprint (con soglie diverse) — o aspetta il blueprint dedicato 😉

## 🔄 Aggiornamenti

Quando pubblichiamo una nuova versione: *Impostazioni → Automazioni e scene → Progetti → menu ⋮ sul progetto → **Reimporta***. Le automazioni già create si aggiornano da sole.

## ❤️ Supporta il progetto

Il blueprint è gratuito. Se ti fa risparmiare: [offrici un caffè via PayPal](https://www.paypal.com/ncp/payment/CD3C2UMCK8J7L), visita lo [store](https://dinamotech.it) (codice **DINAMOTECH10**) o iscriviti al [canale](https://www.youtube.com/@DinamoTech).

## 📋 Changelog

- **v1.0** — Prima release: modalità ACS setpoint / ACS resistenza / power station, modello fisico P_max−P_carico, spegnimento automatico a power station carica (con sensore di potenza opzionale), watchdog sensori, blocco con avviso, PV di terze parti opzionale, sicurezza sul prelievo dalla rete, predisposizione al controllo tramite presa di backup della Solarbank.
