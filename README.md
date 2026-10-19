# Documentazione di Progetto: Simulatore ATM e Integrazione

**Autore:** [Il tuo nome]
**Data:** [Data di oggi]

---

## PARTE 1: BUR - Business Requirements

### 1. Obiettivo del Progetto
L'obiettivo è creare un simulatore web di uno sportello Bancomat (ATM) che permetta agli utenti di simulare operazioni bancarie (prelievo, versamento, saldo). Le informazioni di queste operazioni dovranno essere salvate in un formato standard per permettere a un sistema centrale di effettuare controlli di sicurezza e validità.

### 2. Requisiti di Business
* L'utente deve poter inserire il proprio numero di carta.
* L'utente deve poter scegliere l'operazione da effettuare.
* L'utente deve poter specificare un importo.
* Il sistema deve registrare l'ora esatta dell'operazione.
* I dati generati devono essere controllati in modo automatico per prevenire frodi o errori.

---

## PARTE 2: FUD - Analisi Funzionale

### 1. Descrizione del Flusso (Use Case)
1. L'utente accede all'interfaccia web dell'ATM.
2. Compila i campi: Numero Carta, Tipo Operazione, Importo.
3. Preme il tasto "Conferma".
4. Il sistema "Front-end" raccoglie i dati e genera un file formattato (XML).
5. Il file XML viene inviato a un database centrale.
6. Un sistema di integrazione (TIBCO) legge periodicamente i nuovi record dal database per effettuare i controlli.

### 2. Mappatura dei Dati
Il file XML in uscita dovrà contenere i seguenti campi obbligatori:
* **CardNumber:** Alfanumerico, identifica il cliente.
* **OperationType:** Lista di valori chiusa (PRELIEVO, VERSAMENTO, SALDO).
* **Amount:** Valore numerico (con possibili decimali).
* **Timestamp:** Data e ora dell'operazione in formato standard ISO.

### 3. Regole e Controlli (Requisiti per TIBCO)
Il sistema di controllo (TIBCO) dovrà verificare che:
* Il campo *Amount* sia maggiore di zero (tranne che per l'operazione SALDO).
* Il *CardNumber* non sia vuoto o nullo.

---

## PARTE 3: TAD - Analisi Tecnica

### 1. Architettura di Sistema
Il sistema è diviso in tre livelli principali:
* **Front-end (Client):** Applicazione Web scritta in HTML5, CSS3 e JavaScript (Vanilla). Raccoglie l'input dell'utente e genera il payload XML.
* **Storage (Database):** PostgreSQL. Verrà creata una tabella dedicata per ospitare l'XML generato.
* **Middleware / Orchestration:** TIBCO. Effettuerà un *polling* (lettura ciclica) sul database PostgreSQL per prelevare i nuovi file XML e processare le regole di business definite nel FUD.

### 2. Struttura del Database (PostgreSQL)
Nome Tabella prevista: `atm_transactions`
* `id`: SERIAL PRIMARY KEY
* `xml_payload`: XML (contiene l'intero file generato dall'interfaccia)
* `status`: VARCHAR(20) (Default 'NEW' - verrà aggiornato da TIBCO in 'PROCESSED' o 'ERROR')
* `created_at`: TIMESTAMP DEFAULT CURRENT_TIMESTAMP

### 3. Struttura del Tracciato Record (XML Schema)
L'output generato dalla funzione JavaScript `generateXML()` rispetta la seguente struttura:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Transaction>
    <CardNumber>String</CardNumber>
    <OperationType>String</OperationType>
    <Amount>Decimal</Amount>
    <Timestamp>DateTime</Timestamp>
</Transaction>
```

### 4. Dettagli di Implementazione (Attuale)
Il salvataggio del file attualmente avviene in locale simulando l'export tramite un oggetto `Blob` in JavaScript. Nelle fasi successive, il download locale sarà sostituito da una chiamata di rete (es. API REST) per l'inserimento diretto nel database PostgreSQL.