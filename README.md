# # Tirocinio: Anonimizzazione/pseudonimizzazione di dataset: confronto tecniche e rischio di re-identificazione

In questo progetto utilizziamo i dataset sintetici generati da Synthea per simulare l'anonimizzazione e la pseudonimizzazione di cartelle cliniche, valutando il rischio di re-identificazione. L'obiettivo è bilanciare la protezione dei dati sensibili con la necessità di mantenere il dataset informativo.

**Tecnologie utilizzate:** Python, Jupyter Notebook, Pandas, hashlib, matplotlib.

---

## 1. Classificazione dei Campi (Data Classification)
L'analisi dei file `patients.csv` e `conditions.csv` ha portato alla seguente classificazione e strategia di mitigazione:

* **Identificatori Diretti:** `Id`, `SSN`, `FIRST`, `LAST`, `ADDRESS`, `LAT`, `LON`. 
  * *Azione:* Rimozione (Soppressione) o Pseudonimizzazione tramite Hashing crittografico (SHA-256).
* **Quasi-Identificatori:** `BIRTHDATE`, `ZIP`, `GENDER`. 
  * *Azione:* Generalizzazione (trasformazione in fasce d'età e troncamento del CAP) per raggiungere la *k-anonymity*.
* **Dati Sensibili:** `DESCRIPTION` (Diagnosi medica). 
  * *Azione:* Protezione contro gli attacchi di re-identificazione mantenendo l'utilità statistica.

## 2. Requisiti di Utilità
Il dataset anonimizzato dovrà preservare la validità statistica per consentire:
1. Il calcolo accurato dell'età media dei pazienti affetti da specifiche patologie.
2. L'estrazione delle patologie più frequenti segmentate per sesso e fascia d'età, mantenendo un errore relativo minimo rispetto ai dati in chiaro.

## 3. Analisi del Trade-off: Privacy vs. Utilità (Fase 3)
Abbiamo operato una scelta consapevole che riflette il compromesso tipico della privacy ingegneristica:

* **Protezione (Privacy):** Attraverso tecniche di generalizzazione abbiamo raggiunto un livello di **k-anonymity pari a k=5**. Questo assicura che ogni individuo sia indistinguibile all'interno di un gruppo di almeno 5 persone, annullando il rischio di re-identificazione certa (k=1) presente nel dataset originale.
* **Utilità (Analisi):** Abbiamo validato il dataset anonimizzato confrontando le statistiche pre e post-trasformazione. I risultati mostrano che:
  * Le **metriche aggregate** (come l'età media e la distribuzione di genere) presentano uno scostamento minimo (errore relativo < 1%), confermando che il dataset conserva un'elevata utilità per studi statistici e trend macroscopici.
  * Le **query specifiche e localizzate** subiscono una perdita di precisione maggiore. Questa distorsione è una conseguenza voluta della protezione, poiché impedisce a un attaccante di isolare target geografici specifici.

*Conclusione:* Il dataset ottenuto è considerato "sicuro" per la condivisione a fini di ricerca, in quanto garantisce la protezione delle minoranze demografiche pur mantenendo l'integrità delle analisi statistiche complessive.

## 4. Analisi del Rischio e Raccomandazioni (Fase 4)
Per valutare la robustezza della soluzione, abbiamo simulato un attaccante in possesso di informazioni demografiche parziali (Età, Sesso, CAP). 

* **Impatto delle generalizzazioni:** Allargando le classi di equivalenza (es. passando da fasce d'età di 10 anni a 50 anni), il rischio massimo crolla drasticamente, ma rende il dataset inutilizzabile per fini medici.
* **Linee guida di rilascio (Policy):** È stata definita una soglia rigida di sicurezza. Si raccomanda di non rilasciare alcun dataset in cui il rischio di re-identificazione per una singola classe di equivalenza superi il **20%** (ovvero k < 5).
* **Raccomandazione finale:** L'implementazione attuale (Fasce di 20 anni e CAP a 2 cifre) è risultata l'unica configurazione testata in grado di superare l'audit della policy di rilascio, garantendo la conformità senza distruggere i trend diagnostici.

## 5. Riproducibilità e Demo (Fase 5)
L'intera pipeline di anonimizzazione, la valutazione dell'utilità e l'analisi del rischio sono state consolidate in un unico Jupyter Notebook (`Synthea.ipynb`) completamente riproducibile.

### 5.1 Come eseguire la demo
Il notebook è strutturato per essere eseguito in modo sequenziale ("Run All"). La cella finale contiene una demo interattiva che applica la pipeline automatica testando tre diverse configurazioni per dimostrare come i parametri influenzino il risultato:

* **Parametri configurabili:** La funzione `pipeline_anonimizzazione` accetta in input `fascia_eta` (ampiezza in anni) e `cifre_cap` (numero di cifre in chiaro).
* **Configurazione 1 (Bassa Privacy):** `fascia_eta=10`, `cifre_cap=3`. 
  * *Risultato:* Altissima precisione medica, ma fallisce l'audit di sicurezza (Rischio 100%, k=1).
* **Configurazione 2 (Compromesso Ottimale - Target):** `fascia_eta=20`, `cifre_cap=2`. 
  * *Risultato:* Passa la policy di rilascio garantendo un k=5 (Rischio max 20%) con errori statistici relativi trascurabili (es. 0.09% sull'età media).
* **Configurazione 3 (Alta Privacy):** `fascia_eta=50`, `cifre_cap=1`. 
  * *Risultato:* Sicurezza estrema (k=35), ma distrugge il valore del dataset per l'analisi medica.
