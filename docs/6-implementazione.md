# Aspetti implementativi
## Camunda - Java
La parte seguente attiene all'implementazione dei vari oggetti presenti in camunda. 

Il diagramma BPMN comprende un processo per Acmesky e un altro per il cliente.

Le altre componenti (prontogram, servizio di pagamento ecc.) sono state realizzate tramite server python, al netto del servizio di trasporto, che è stato realizzato in jolie, come da specifiche.

Il processo ha due starting point indipendenti: l'invio di un messaggio al processo da parte del cliente, oppure la scadenza del timer presente in acmesky che avvia la richiesta giornaliera delle offerte. 

### Invio interessi cliente
È possibile inviare due tipi di messaggi al processo di Camunda:
- un messaggio contenente un interesse dell'utente (composto da nome utente, indirizzo di casa, data di partenza, aeroporto di partenza e di arrivo e, opzionalmente, data di ritorno);
- un messaggio vuoto che comporta la generazione automatica di un interesse con parametri casuali. 

Viene inoltre generato un oggetto della classe ```Transazione```, il quale contiene tutti i dati utili per completare una transazione, quali nome utente, codice dell'offerta di acmesky, link di pagamento etc.: per *transazione* si intendono tutti i processi che intercorrono tra l'invio/generazione dell'interesse e la ricezione del biglietto acquistato. 

Successivamente l'utente si mette in attesa di un ack da parte di acmesky per confermare il ricevimento dell'interesse: in caso contrario, l'istanza di processo termina dopo 1 minuto. 

### Richiesta dei voli
All'avvio del processo relativo ad Acmesky, ecco che Acmesky legge una serie di file ```.list```, contenenti la lista dei server (ad esempio, ```airline.list``` contiene la lista di tutte le compagnie aeree; i nomi sono autoesplicativi).

In questo modo, se un server cambia indirizzo, è sufficiente modificare il file e alla successiva creazione di un'istanza del processo di Acmesky, quest'ultimo avrà direttamente la lista aggiornata dei server disponibili.

Il processo procede col chiedere a ogni compagnia aerea le proprie offerte; al termine di questa fase, acmesky inizia a cercare i match con gli interessi degli utenti.\
Come accennato sopra, la ricerca di match la si ha anche a seguito della ricezione di un interesse da parte di un utente.

### Ricerca dei match
Acmesky cerca, per ogni utente, e per ogni interesse associato all'utente, un volo compatibile: in caso l'utente sia interessato tanto al volo di andata quanto a quello di ritorno e non siano disponibili entrambi, il suo interesse non viene considerato soddisfatto, per cui non si genera alcun match. 

Viene quindi creata una lista dei match; per ogni match, viene poi generato un codice che Acmesky invierà a Prontogram, che, a sua volta, provvederà ad inoltrarlo all'utente: in caso l'utente stia cercando sia volo di andata che di ritorno per cui vi è match, riceverà due codici. 

#### Prontogram
Sebbene sarebbe più realistico che un cliente abbia a disposizione un client che si colleghi a prontogram (ad esempio tramite polling) che gli notifichi la presenza di un nuovo messaggio, si è scelto di fare in modo 
che sia prontogram a inviare un messaggio al processo del cliente, mediante le api di Camunda. 

### Risposta del cliente al codice offerta inviato da Acmesky
L'utente può scegliere se accettare l'offerta o meno (abbiamo settato una probabilità del 20% che ciò non accada).

Ricevuto il codice dall'utente, Acmesky ne controlla la validità - in tempo ed esistente - e crea un oggetto di tipo `Transazione` (che potrà cancellare solo alla fine) contenente i dati utili per completare il processo (indirizzo dell'utente, codice offerta, ...). 

A questo punto, Acmesky richiede ai fornitori dei servizi bancari la generazione di un link di pagamento che quindi lo inoltra al cliente; se il cliente effettua il pagamento con successo, Acmesky ne viene messo a conoscenzan e procede col controllare le seguenti due condizioni: 
1. il pagamento supera 1000€/$;
2. l'utente si trova a meno di 30 km dall'aeroporto di interesse.

Il punto 2. viene effettuato interrogando il servizio del calcolo delle distanze, il quale a sua volta si appoggia alle API di Google Maps (*reverse geocoding*). 

Se le due condizioni sono soddisfatte, Acmesky propone all'utente il servizio di trasporto, il quale va a rifiutare con una probabilità del 10%.

### Termine del processo e invio dell'offerta di trasporto
Il servizio di pagamento invia una notifica alla compagnia aerea circa il successo della transazione; la compagnia aerea invia allora un "biglietto" all'utente (in realtà, a livello implementativo, è un semplice messaggio contente il nome utente e i dati associati al volo, inviato sempre mediante le API di Camunda).
Il cliente può allora rimuovere la transazione associata dalla lista delle transazioni attive.

Nel caso acmesky non debba inviare all'utente l'offerta di trasporto navetta, può rimuovere la transazione in corso dalla lista delle transazioni attive; altrimenti, procede inviando all'utente la proposta di utilizzo navetta; se l'utente accetta, acmesky contatta la compagnia dei trasporti più vicina (informazione ottenuta interrogando il servizio del calcolo delle distanze).\
Successivamente, per effettuare la prenotazione, acmesky deve inviare dei dati alla compagnia, come l'indirizzo dell'utente e la data di richiesta del trasporto; notare che la compagnia richiederebbe l'indirizzo diviso nelle varie componenti, ma viene inviato solo via e città per motivi di semplicità, in quanto non è stato implementato un parsing dell'indirizzo. Per tale motivo, Acmesky richiede che l'utente invii solo via e città - onde evitare di generare tutti i campi necessari (non sono necessari per la dimostrazione dell'esecuzione del processo, si evita di abusare delle api di google maps per controllare la validità dei dati, essendo a pagamento). 

Al termine della prenotazione Acmesky può rimuovere l'offerta dalla lista delle offerte attive.