# Diagramma BPMN

Il modello BPMN cui siamo giunti è qui riportato:
![Diagramma BPMN](
 "Diagramma BPMN")

Per una maggiore chiarezza, articoliamo anche la trattazione del diagramma BPMN nelle medesime 4 parti individuate a partire dalla spiegazione della coreografia:
- Registrazione degli interessi degli utenti;
- Richiesta delle offerte e inoltro;
- Ricezione delle offerte LM e inoltro;
- Acquisto di un'offerta.

## Registrazione interessi degli utenti
Questa parte attiene alle seguenti attività:
1. L'utente invia i propri interessi ad ACMESky: in caso ACMESky entro un minuto non gli notifichi il salvataggio riuscito degli interessi che ha inviato, ritenta l'invio per un certo numero di tentativi;
2. ACMESky salva gli interessi ricevuti e notifica il salvataggio riuscito al cliente.

[![Diagramma BPMN relativo alla richiesta e all'inoltro delle offerte](https://github.com/MickPerl/soseng-project-documentation/blob/master/assets/images/UML_richiesta_inoltro.png?raw=true
 "Diagramma BPMN relativo alla richiesta e all'inoltro delle offerte")](https://github.com/MickPerl/soseng-project-documentation/blob/master/assets/images/UML_richiesta_inoltro.png?raw=true)


## Richiesta delle offerte e inoltro
Questa parte attiene alle seguenti attività:
1. ACMESky richiede alle compagnie aeree le offerte attive;
2. ACMESky memorizza le offerte ricevute;
3. ACMESky ricerca corrispondenze tra le offerte memorizzate e gli interessi ricevuti dagli utenti: per ogni corrispondenza trovata, genera un codice identificativo della coppia offerta-utente;
4. ACMESky invia a Prontogram le offerte (descrizione + codice) che incontrano gli interessi di qualche utente;
5. Prontogram inoltra le offerte agli utenti corrispondenti.



## Ricezione delle offerte LM e inoltro
Questa parte attiene alle seguenti attività:
1. Le compagnie aeree inviano ad ACMESky le offerte last-minute non appena le attivano;
2. ACMESky memorizza le offerte LM ricevute;
3. Punti 3-4-5 della sezione "Richiesta delle offerte e inoltro".

## Acquisto offerta
Questa parte attiene alle seguenti attività:
1. l'utente invia il codice dell'offerta che intende acquistare ad Acmesky e si mette in attesa di una comunicazione da Acmesky conseguente al controllo che realizzerà sul codice: Acmesky può notificargli che il codice è errato, caso in cui l'utente ritenta l'invio del codice, o il link a cui effettuare il pagamento;
2. dopo aver inviato il pagamento al link del fornitore dei servizi bancari che gli è stato indicato, l'utente si mette in attesa di una comunicazione di quest'ultimo: in caso tale comunicazione notifichi che il pagamento è fallito (segue tale comunicazione anche ad Acmesky), l'utente lo ritenta, diversamente ne prende atto;
3. il fornitore dei servizi bancari invia le quote spettanti ad Acmesky e alla compagnia aerea interessata, la quale procede quindi ad inviare all'utente il biglietto acquistato;
4. dopo aver ricevuto la parte che gli spetta dal fornitore dei servizi bancari, Acmesky contrassegna il codice offerta-cliente acquistato come utilizzato;
5. Acmesky controlla il prezzo dell'offerta acquistata: nel caso superi le 1000 euro, verifica, servendosi delle API di GMaps, che la casa dell'utente disti dall'aeroporto di interesse non più di 30 km, nel caso in cui chiede all'utente se vuole il servizio di trasporto con la navetta;
6. nel caso l'utente risponda affermativamente, Acmesky, servendosi delle API di GMaps, raccoglie le distanze tra le compagnie di trasporto e il domicilio dell'utente, per poi prenotare il servizio di trasporto presso la compagnia più vicina.<>
