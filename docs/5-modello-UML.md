# Modello UML della SOA progettata

Per quanto riguarda questa sezione, abbiamo analizzato minuziosamente il diagramma BPMN presentato nella [sezione precedente](4-diagramma-BPMN.md), nel tentativo di individuare ogni interfaccia e le operazioni che ciascuna di esse rende accessibile: abbiamo indicato anche le capability cui queste interfacce permettono di fruire.

A fini documentativi, abbiamo utilizzato UML e, più precisamente, il profilo *TinySOA*, in quanto mette a disposizione gli artefatti e gli stereotipi appropriati per un'architettura orientata ai servizi.

Gli artefatti in questione sono:
- **Task**: un intero processo legato al dominio;
- **Entity**: una singola attività che può far uso di utility;
- **Utility**: un intero processo scollegato dal dominio

Il modello UML cui siamo giunti è qui riportato:
![Modello UML con profilo TinySOA](https://github.com/MickPerl/soseng-project-documentation/blob/master/assets/images/UML_totale.png?raw=true
 "Modello UML dell'architettura totale")

Per una maggiore chiarezza, articoliamo anche la trattazione sul modello UML nelle medesime 4 parti individuate a partire dalla spiegazione della coreografia:
- Registrazione degli interessi degli utenti;
- Richiesta delle offerte e inoltro;
- Ricezione delle offerte LM e inoltro;
- Acquisto del biglietto.

<!-- theme: warning -->
> **Convenzioni adottate nella formattazione**:
> - `capability`
> - **interfaccia**
> - *operazione*

## Registrazione interessi degli utenti
![Modello UML con profilo TinySOA](https://github.com/MickPerl/soseng-project-documentation/blob/master/assets/images/UML_registra_interessi.png?raw=true
 "Modello UML relativo alla registrazione degli interessi degli utenti")

In questa parte, le capability essenziali sono quelle di ACMESky:
- `MemorizzazioneInteressi` rende persistenti gli interessi registrati ed è esposta dall'interfaccia **salvataggioInteressiInterno**, la quale presenta l'operazione *inviaInteressi* che viene invocata internamente da ACMESky per rendere persistenti gli interessi;
- `RegistrazioneInteressi` realizza la registrazione degli interessi dell'utente ed è esposta dall'interfaccia **invioInteressi**, la quale presenta l'operazione *inviaInteressi* che viene invocata dall'utente per specificare i propri interessi;
  - questa capability necessita delle interfacce **salvataggioInteressiInterno** di cui sopra e **invioAck** del Cliente, in quanto, per finalizzare la registrazione degli interessi, ne deve notificare all'utente l'esito della ricezione e deve renderli persistenti.

## Richiesta e inoltro delle offerte
![Modello UML con profilo TinySOA](https://github.com/MickPerl/soseng-project-documentation/blob/master/assets/images/UML_richiesta_inoltro.png?raw=true
 "Modello UML relativo alla richiesta e all'inoltro delle offerte")
Questa parte attiene alle seguenti attività:
1. la richiesta delle offerte attive alle compagnie aeree da parte di ACMESky;
2. la ricezione e il salvataggio delle offerte da parte di ACMESky;
3. la ricerca di corrispondenze tra le offerte e gli interessi degli utenti;
4. l'invio delle offerte che incontrano gli interessi di qualche utente a Prontogram da parte di ACMESky;
5. l'inoltro delle offerte agli utenti interessati da parte di Prontogram.

Come già esplicitato nella sezione delle coreografie, per ogni offerta, ACMESky invia a Prontogram una *descrizione*, il/i *cliente/i cui è indirizzata* e tanti *codici* quanti sono i clienti destinatari, ognuno dei quali individua univocamente l'offerta e il cliente.

Le capability coinvolte sono:
- `GestioneInteressiOfferte` è la capability di ACMESky, consistente nella ricezione e memorizzazione delle offerte ricevute dalle compagnie aeree, nella ricerca di match con gli interessi specificati dagli utenti e nella generazione di un codice per identificare la coppia offerta-utente: in dettaglio, le operazioni alla base sono esposte da due interfacce diverse, **InvioOfferte** che rende accessibile dall'esterno l'operazione *inviaOfferte* e **InteressiOfferteInterno** che rende accessibile internamente le operazioni *salvaOfferte*, *cercaMatchOfferteInteressi* e *generaCodici*;
  - dipende dalle interfacce **InoltroCodice** di Prontogram e **RichiestaOfferte** della compagnia aerea, in quanto, per finalizzarsi, deve richiedere le offerte alle compagnie aeree, invocando l'operazione *richiestaOfferte* dall'interfaccia **RichiestaOfferte** che espone la capability `RestituzioneOfferte` delle compagnie aeree, e deve inoltrare i codici generati ai clienti corrispondenti mediante l'interfaccia di Prontogram;  
- `InoltroCodice` è la capability di Prontogram tramite cui inoltra agli specifici clienti i codici ricevuti da ACMESky ed è esposta dall'interfaccia **InoltroCodice**, la quale presenta l'operazione *inoltraCodice* che viene invocata da ACMESky per rendere avviare l'inoltro dei codici ai clienti corrispondenti;
  - dipende dall'interfaccia **InvioDati** del cliente in quanto, per finalizzarsi, deve invocare l'operazione *inviaCodice* tramite cui invia il codice al cliente.

## Ricezione e inoltro delle offerte LM
Questa parte, ai fini del modello UML in oggetto, è ricompresa nella parte precedente in quanto, le compagnie aeree che intendono comunicare delle offerte last minute ad ACMESky invocano l'operazione *inviaOfferte* dell'interfaccia **InvioOfferte** di ACMESky, ossia la stessa operazione invocata quando le compagnie aeree restituiscono le offerte attive on demand.

Per distinguere un'offerta last minute da una ordinaria, abbiamo previsto nell'operazione *inviaOfferte* un campo booleano "LM": se la compagnia aerea nell'invocarla la setta a True allora invia delle offerte che saranno considerate da ACMESky come last minute, diversamente sono considerate ordinarie. 

## Acquisto offerta

![Modello UML con profilo TinySOA](https://raw.githubusercontent.com/MickPerl/soseng-project-documentation/master/assets/images/UML_acquista_offerta.png
 "Modello UML relativo alla richiesta e all'inoltro delle offerte")

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

Le capability coinvolte sono:
- `ControlloCodiceEPagamento` di ACMESky che le permette di ricevere il codice dall'utente, verificarne la validità, ricevere il link di pagamento dal fornitore di servizi bancari e ricevere la notifica dell'esito del pagamento; è esposta da diverse interfacce, quali **InvioCodice** che rende accessibile l'operazione *inviaCodice*, **ControlloCodiciEPagamentoInterno** che rende accessibile internamente *controlloCodice* e **InvioDaBanca** che rende accessibile *inviaLinkBanca*, *notificaSuccessoPagameto* e *notificaFallimentoPagamento*;
  - dipende dalle interfacce **ServiziBancari** del Fornitore dei servizi bancari, in quanto ACMESky deve richiedere a questi ultimi la generazione del link di pagamento, **InvioDati** del cliente, in quanto deve inviargli il link di pagamento e **InvioAck** del cliente per comunicargli una notifica per l'esaurimento o incorrettezza del codice inviato;
- `GestionePrenotazioneNavetta` di ACMESky che le permette di ricevere le distanze richieste e l'esito della proposta al cliente del servizio di traporto; è esposta dall'interfaccia **GestionePrenotazioneNavetta** che rende accessibile le operazioni *inviaDistanza*, *inviaAccettazioneTrasporto* e *inviaRifiutoTrasporto*;
  - dipende dalle interfacce **CalcoloDistanze** del FornitoreDistanze che espone la capability `CalcoloDistanza`, in quanto deve richiedergli le distanze di interesse invocando *calcolaDistanza*, **InvioPropostaTrasporto** del cliente, in quanto deve potergli inviargli la proposta del servizio e **Prenotazione** della compagnia di traporto che espone `Prenotazione`, in quanto deve poter prenotarne il servizio, invocando *prenota*;
- `GestionePagamento` di Fornitore dei servizi bancari che gli permette di generare il link di pagamento richiestogli da ACMESky e riceve il pagamento dal cliente; è esposta dall'interfaccia **ServiziBancari** che presenta le operazioni *inviaRichiestaGenerazioneLink* e *inviaPagamento*;
  - dipende dalle interfacce **ServiziBancariInterno**, in quanto deve verificare i dati di pagamento invocando *verificaDati* e **InvioPagamento** della compagnia aerea, in quanto deve potergli recapitare la quota corrispondente;
- `VerificaDatiPagamento` del fornitore dei servizi bancari che gli permette di verificare la validità dei dati di pagamento; è esposta da **ServiziBancariInterno** che presenta *verificaDati*;
  - dipende dall'interfaccia **InvioAck** del cliente, in quanto deve notificargli l'esito del pagamento o l'esaurimento dei tentativi;
- `RicezionePagamentoInvioBiglietto` della compagnia aerea che le permette di ricevere la quota del pagamento che gli spetta dal fornitore dei servizi bancari; è esposta da **InvioPagamento** che presenta *inviaPagamento*;
  - dipende da **InvioBiglietto** del cliente, in quanto deve potergli inviare il biglietto, invocando l'operazione *inviaBiglietto*.