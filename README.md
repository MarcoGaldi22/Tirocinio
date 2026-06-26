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
