# Diagramma BPMN

Il modello UML cui siamo giunti è qui riportato:
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
Assunzione mezzi compagnie trasporti infiniti



Questa parte attiene alle seguenti attività:
1. la ricezione del codice di ACMESky e il controllo della sua validità;
2. la richiesta del link di pagamento al fornitore dei servizi bancari da parte di ACMESky e la sua ricezione di quest'ultimo;
3. l'invio del link di pagamento al cliente da parte di ACMESky;
4. l'invio dei dati di pagamento al fornitore dei servizi bancari da parte del cliente
5. la verifica dei dati di pagamento ad opera del fornitore dei servizi bancari;
6. l'invio della notifica circa l'esito del pagamento ai clienti da parte dei fornitori dei servizi bancari;
7. l'invio dei fornitori dei servizi bancari della quota del pagamento ad ACMESky;
8. l'invio dei fornitori dei servizi bancari della quota del pagamento alla compagnia aerea;
9. l'invio del biglietto al cliente da parte della compagnia aerea;
10. la richiesta di ACMESky al fornitore delle distanze circa la distanza tra il domicilio dell'utente e l'aeroporto di partenza dell'aereo di suo interesse;
11. la proposta del serivizio di trasporto al cliente da parte di ACMESky e la risposta del cliente;
12. in caso l'utente accetti, la richiestadi ACMESky al fornitore delle distanze circa le distanze tra il domicilio dell'utente e le sedi delle compagnie di trasporto;
13. la prenotazione di ACMESky di un servizio di trasporto;  
