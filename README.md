# Tirocinio: Anonimizzazione Dataset Medico (Synthea)

## Fase 1: Analisi e Classificazione dei Dati
In questo progetto utilizziamo i dataset sintetici generati da **Synthea** per simulare l'anonimizzazione e la pseudonimizzazione di cartelle cliniche, valutando il rischio di re-identificazione.

### 1. Classificazione dei Campi (Data Classification)
L'analisi dei file `patients.csv` e `conditions.csv` ha portato alla seguente classificazione e strategia di mitigazione:
* **Identificatori Diretti:** `Id`, `SSN`, `FIRST`, `LAST`, `ADDRESS`, `LAT`, `LON`. 
  * *Azione:* Rimozione (Soppressione) o Pseudonimizzazione tramite Hashing crittografico (SHA-256).
* **Quasi-Identificatori:** `BIRTHDATE`, `ZIP`, `GENDER`. 
  * *Azione:* Generalizzazione (trasformazione in fasce d'età e troncamento del CAP) per raggiungere la k-anonymity.
* **Dati Sensibili:** `DESCRIPTION` (Diagnosi medica). 
  * *Azione:* Protezione contro gli attacchi di re-identificazione.

### 2. Requisiti di Utilità
L'obiettivo è raggiungere un compromesso ottimale tra privacy e utilità. Il dataset anonimizzato dovrà preservare la validità statistica per consentire:
* Il calcolo accurato dell'età media dei pazienti affetti da specifiche patologie.
* L'estrazione delle patologie più frequenti segmentate per sesso e fascia d'età, mantenendo un errore relativo minimo rispetto ai dati in chiaro.


## 3. Analisi del Trade-off: Privacy vs. Utilità (Fase 3)
L'obiettivo di questo progetto è bilanciare la protezione dei dati sensibili con la necessità di mantenere il dataset informativo. Abbiamo operato una scelta consapevole che riflette il compromesso tipico della privacy ingegneristica:

* **Protezione (Privacy):** Attraverso tecniche di generalizzazione abbiamo raggiunto un livello di **k-anonymity pari a k=5**. Questo assicura che ogni individuo sia indistinguibile all'interno di un gruppo di almeno 5 persone, annullando il rischio di re-identificazione certa (k=1) presente nel dataset originale.
* **Utilità (Analisi):** Abbiamo validato il dataset anonimizzato confrontando le statistiche pre e post-trasformazione. I risultati mostrano che:
    * Le **metriche aggregate** (come l'età media e la distribuzione di genere) presentano uno scostamento minimo (errore relativo < 1%), confermando che il dataset conserva un'elevata utilità per studi statistici e trend macroscopici.
    * Le **query specifiche e localizzate** subiscono una perdita di precisione maggiore. Questa distorsione è una conseguenza voluta della protezione, poiché impedisce a un attaccante di isolare target geografici specifici.

**Conclusione:** Il dataset ottenuto è considerato "sicuro" per la condivisione a fini di ricerca, in quanto garantisce la protezione delle minoranze demografiche pur mantenendo l'integrità delle analisi statistiche complessive.

## 4. Analisi del Rischio e Raccomandazioni (Fase 4)
Per valutare la robustezza della soluzione, abbiamo simulato un attaccante in possesso di informazioni demografiche parziali (Età, Sesso, CAP) valutando l'impatto di tre diverse configurazioni di generalizzazione.

* **Impatto delle generalizzazioni:** Allargando le classi di equivalenza (es. passando da fasce d'età di 10 anni a 50 anni), il rischio massimo crolla drasticamente, ma rende il dataset inutilizzabile per fini medici. 
* **Linee guida di rilascio (Policy):** È stata definita una soglia rigida di sicurezza. Si raccomanda di **non rilasciare** alcun dataset in cui il rischio di re-identificazione per una singola classe di equivalenza superi il **20%** (ovvero $k < 5$). 
* **Raccomandazione finale:** L'implementazione attuale (Fasce di 20 anni e CAP a 2 cifre) è risultata l'unica configurazione testata in grado di superare l'audit della policy di rilascio, garantendo la conformità senza distruggere i trend diagnostici.
