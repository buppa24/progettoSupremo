# Documentazione Progetto: ATM Bancomat su TIBCO BW 6

**Autore:** [Il tuo nome]
**Data:** [Data di oggi]

---

## 1. BUR (Business Requirements)

**Richieste del business al tecnico**

Il cliente richiede la realizzazione di un'interfaccia software per uno sportello ATM. Il sistema deve gestire esclusivamente la funzione di **prelievo**. 

Per l'autenticazione, il sistema non utilizzerà il numero di carta, ma dovrà riconoscere il cliente tramite l'inserimento dei seguenti dati anagrafici:
* Nome
* Cognome
* PIN univoco di 5 cifre

Per quanto riguarda l'operatività, l'ATM deve permettere all'utente di selezionare l'importo da prelevare (verificando idealmente la compatibilità con il saldo). La selezione dell'importo deve avvenire tramite dei radio button per i tagli standard (50€, 100€, 200€) e prevedere anche un campo di testo per l'inserimento di un importo libero.

Al termine dell'operazione, all'invio della richiesta, il sistema deve mostrare un popup di avviso con il messaggio esatto: *"prelevare il denaro entro 30 secondi"*.
L'applicativo deve essere sviluppato come pagina HTML. Tramite uno script JavaScript, i dati inseriti devono essere formattati in un record XML e salvati fisicamente come file all'interno di una cartella di appoggio.

---

## 2. FUD (Analisi Funzionale)

**Analisi e comprensione dei requisiti**

Sulla base delle richieste del cliente, il sistema prevederà un'unica schermata (front-end) in cui l'utente inserirà i dati necessari per completare la transazione di prelievo.

**Flusso operativo (Use Case principale):**
1. L'utente visualizza la maschera dell'ATM.
2. L'utente compila i campi di riconoscimento: `Nome`, `Cognome`, `PIN` (quest'ultimo deve accettare solo 5 caratteri numerici).
3. L'utente seleziona l'importo desiderato scegliendo tra le opzioni predefinite (radio buttons: 50, 100, 200) oppure inserendo un valore numerico nel campo libero.
4. L'utente preme il pulsante di conferma prelievo.
5. Il sistema mostra immediatamente il popup alert: *"prelevare il denaro entro 30 secondi"*.
6. Contestualmente, il sistema genera un file XML contenente i dati della transazione e lo deposita in una directory specifica, da cui poi TIBCO BW 6 andrà a prelevarlo per le elaborazioni successive.

**Tracciato Dati Previsto:**
* `Nome`: Stringa di testo.
* `Cognome`: Stringa di testo.
* `PIN`: Numerico (esattamente 5 cifre).
* `Importo`: Valore numerico intero.

---

## 3. TAD (Analisi Tecnica)

**Dettagli implementativi per gli sviluppatori**

Il progetto è composto da un'interfaccia client-side e da un flusso di integrazione gestito tramite middleware.

**Stack Tecnologico Frontend:**
* **HTML5 / CSS:** Per la strutturazione e la formattazione della pagina (form di input, radio buttons, campi text/number).
* **JavaScript (Vanilla):** Gestisce la logica del form. Si occupa di validare la lunghezza del PIN (5 cifre), capire se l'utente ha usato un radio button o il campo libero, mostrare il `window.alert()` richiesto e generare il file XML. Per il salvataggio del file su cartella, JavaScript sfrutterà le API del browser (oggetto `Blob` e URL dinamico) forzando il download del file `.xml` nella cartella di default, simulando così la scrittura su filesystem.

**Middleware (TIBCO BW 6):**
Il processo di TIBCO BusinessWorks 6 verrà configurato con un componente di tipo *File Poller*. Questo ricevitore monitorerà la cartella di destinazione dei file generati dall'HTML. Non appena rileverà la presenza di un nuovo file XML, lo prenderà in carico, ne effettuerà il parsing e applicherà la logica di business (es. validazione pin, verifica capienza saldo simulata, instradamento verso altri sistemi).

**Struttura XML Generata (Esempio):**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<RichiestaPrelievo>
    <Cliente>
        <Nome>Mario</Nome>
        <Cognome>Rossi</Cognome>
        <PIN>12345</PIN>
    </Cliente>
    <DatiOperazione>
        <Tipo>PRELIEVO</Tipo>
        <Importo>100</Importo>
    </DatiOperazione>
</RichiestaPrelievo>
```

---

## 4. Casi di Test

*Di seguito l'estratto dei casi di test (integrato dal foglio Excel di collaudo).*

| ID Test | Descrizione Scenario | Input Inserito | Risultato Atteso | Esito |
| :--- | :--- | :--- | :--- | :--- |
| **TC_01** | Prelievo standard importo fisso | Nome: Mario, Cognome: Rossi, PIN: 12345, Importo: [Radio 50] | Popup 30 sec + Download file XML con importo 50 | |
| **TC_02** | Prelievo con importo libero | Nome: Luigi, Cognome: Verdi, PIN: 99887, Importo: [Libero 150] | Popup 30 sec + Download file XML con importo 150 | |
| **TC_03** | Validazione PIN errato (meno di 5 cifre) | Nome: A., Cognome: B., PIN: 123 | Il sistema blocca l'invio e chiede un PIN di 5 cifre | |
| **TC_04** | Validazione formato XML | Esecuzione di TC_01 | Il file salvato deve rispettare la struttura concordata nel TAD | |