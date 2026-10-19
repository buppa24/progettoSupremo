# TAD - Analisi Tecnica
**Progetto:** Architettura Simulatore ATM, DB e TIBCO
**Autore:** [Il tuo nome / Progettista Tecnico]

## 1. Architettura di Sistema
Il sistema è diviso in tre livelli principali:
* **Front-end (Client):** Applicazione Web scritta in HTML5, CSS3 e JavaScript. Si occupa di raccogliere l'input dell'utente e generare il payload XML.
* **Storage (Database):** PostgreSQL. Verrà creata una tabella per ospitare l'XML in formato testuale o nel tipo di dato nativo `XML`.
* **Middleware / Orchestration:** TIBCO. Effettuerà un *polling* (lettura ciclica) sul database PostgreSQL per prelevare i nuovi file XML e processare le regole di business.

## 2. Struttura del Database (PostgreSQL)
Nome Tabella: `atm_transactions`
* `id`: SERIAL PRIMARY KEY
* `xml_payload`: XML (contiene l'intero file generato dall'interfaccia)
* `status`: VARCHAR(20) (Default 'NEW' - verrà aggiornato in 'PROCESSED' o 'ERROR')
* `created_at`: TIMESTAMP DEFAULT CURRENT_TIMESTAMP

## 3. Struttura del Tracciato Record (XML Schema)
L'output XML generato rispetterà la seguente struttura:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Transaction>
    <CardNumber>String</CardNumber>
    <OperationType>String</OperationType>
    <Amount>Decimal</Amount>
    <Timestamp>DateTime</Timestamp>
</Transaction>
```

## 4. Dettagli di Implementazione JavaScript
Attualmente, il salvataggio genera un file locale per simulare l'output. Nella versione finale, verrà implementata una chiamata API verso il backend per effettuare l'inserimento nel database PostgreSQL.