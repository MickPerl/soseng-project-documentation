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

## Registrazione interessi degli utenti
![Modello UML con profilo TinySOA](https://github.com/MickPerl/soseng-project-documentation/blob/master/assets/images/UML_registra_interessi.png?raw=true
 "Modello UML relativo alla registrazione degli interessi degli utenti")

In questa parte, le capability essenziali sono quelle di ACMESky:
- `MemorizzazioneInteressi` rende persistenti gli interessi registrati  ed è esposta dall'interfaccia **salvataggioInteressiInterno**, la quale presenta l'operazione *inviaInteressi* che viene invocata internamente da ACMESky per rendere persistenti gli interessi;
- `RegistrazioneInteressi` realizza la registrazione degli interessi dell'utente ed è esposta dall'interfaccia **invioInteressi**, la quale presenta l'operazione *inviaInteressi* che viene invocata dall'utente per specificare i propri interessi;
  - questa capability necessita delle interfacce **salvataggioInteressiInterno** di cui sopra e **invioAck** del Cliente, in quanto, per finalizzare la registrazione degli interessi, ne deve notificare all'utente l'esito della ricezione e deve renderli persistenti.

## Richiesta e inoltro delle offerte
![Modello UML con profilo TinySOA](https://github.com/MickPerl/soseng-project-documentation/blob/master/assets/images/UML_richiesta_inoltro.png?raw=true
 "Modello UML relativo alla richiesta e all'inoltro delle offerte")
Questa parte attiene alle seguenti attività:
- la richiesta delle offerte attive alle compagnie aeree da parte di ACMESky;
- la ricerca di corrispondenze tra le offerte e gli interessi degli utenti;
- l'invio delle offerte che incontrano gli interessi di qualche utente a Prontogram da parte di ACMESky;
- l'inoltro delle offerte agli utenti interessati da parte di Prontogram.

Come già esplicitato nella sezione delle coreografie, per ogni offerta, ACMESky invia a Prontogram una *descrizione*, il/i *cliente/i cui è indirizzata* e tanti *codici* quanti sono i clienti destinatari, ognuno dei quali individua univocamente l'offerta e il cliente.

Le capability coinvolte sono:
- `GestioneCorrispondenzaOfferteInteressi` cerca match tra le offerte rivevute dalle compagnie aeree e : in dettaglio, le operazioni necessarie a realizzarla sono esposte da due interfacce diverse, **d** accessibile dall'esterno e **d**
  - dipende dalle interfacce **InoltroCodice** di Prontogram e **RichiestaOfferte** della compagnia aerea, in quanto, per finalizzarsi, deve inoltrare 

## Ricezione e inoltro delle offerte LM
Uguale a prima

L'operazione 
Un'offerta last minute si distingue da una ordinaria per i seguenti aspetti:
- l'offerta  



## Acquisto offerta

![Modello UML con profilo TinySOA](https://raw.githubusercontent.com/MickPerl/soseng-project-documentation/master/assets/images/UML_acquista_offerta.png
 "Modello UML relativo alla richiesta e all'inoltro delle offerte")


 