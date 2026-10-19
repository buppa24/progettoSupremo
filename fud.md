# FUD - Analisi Funzionale
**Progetto:** Simulatore ATM - Flusso Generazione XML
**Autore:** [Il tuo nome / Analista Funzionale]

## 1. Descrizione del Flusso (Use Case)
1. L'utente accede all'interfaccia web dell'ATM.
2. Compila i campi: Numero Carta, Tipo Operazione, Importo.
3. Preme il tasto "Conferma".
4. Il sistema "Front-end" raccoglie i dati e genera un file formattato (XML).
5. Il file XML viene inviato a un database centrale.
6. Un sistema di integrazione (TIBCO) legge periodicamente i nuovi record dal database.

## 2. Mappatura dei Dati
Il file XML in uscita dovrà contenere i seguenti campi obbligatori:
* **CardNumber:** Alfanumerico, identifica il cliente.
* **OperationType:** Lista di valori chiusa (PRELIEVO, VERSAMENTO, SALDO).
* **Amount:** Valore numerico (con possibili decimali).
* **Timestamp:** Data e ora dell'operazione in formato standard ISO.

## 3. Regole e Controlli
Il sistema di controllo (TIBCO) dovrà verificare che:
* Il campo *Amount* sia maggiore di zero (tranne che per l'operazione SALDO).
* Il *CardNumber* non sia vuoto.