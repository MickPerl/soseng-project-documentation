# Coreografia e proiezioni

Abbiamo modellato le comunicazioni dello scenario in oggetto definendo una coreografia: in dettaglio, per la sua definizione formale ci siamo serviti di un *process calculi*, essendo più accurato di WS-CDL (Web Services Choreography Description Language) e della formalizzazione per le coreografie di BPMN.

Abbiamo poi verificato le condizioni di connectedness e quindi proiettato l'output in un sistema di ruoli.

## Coreografia
La coreografia cui siamo giunti dopo numerosi confronti, riflessioni e iterazioni è presentata di seguito: 

```javascript
C ::= (invio_interesse: cliente --> acmesky)*

|

(((richiesta_offerte: acmesky --> compagnia_aerea_a ;
risposta_offerte: compagnia_aerea_a --> acmesky) | ... |
(richiesta_offerte: acmesky --> compagnia_aerea_z ; 
risposta_offerte: compagnia_aerea_z --> acmesky)) ; 
1 + (invio_codici: acmesky --> prontogram ;
(inoltro_codice: prontogram --> cliente_1 ; ... ; 
inoltro_codice: prontogram --> cliente_n)))

|     

((invio_offerte_LM: compagnia_aerea_a --> acmesky ;
1 + (invio_codici_LM: acmesky --> prontogram ; 
(inoltro_codice_LM: prontogram --> cliente_1a ; ... ;
inoltro_codice_LM: prontogram --> cliente_na))) | 
... |
(invio_offerte_LM: compagnia_aerea_z --> acmesky ;
1 + (invio_codici_LM: acmesky --> prontogram ; 
(inoltro_codice_LM: prontogram --> cliente_1z ; ... ; 
inoltro_codice_LM: prontogram --> cliente_nz)) ))

| 

(invio_codice_offerta: cliente --> acmesky ;
(notifica_errore_codice: acmesky --> cliente) +
(richiesta_link_pagamento: acmesky --> fornitore_servizi_bancari ;
invio_link_pagamento: fornitore_servizi_bancari --> acmesky;
inoltro_link_pagamento: acmesky --> cliente ;
pagamento: cliente --> fornitore_servizi_bancari ;
(notifica_errore_pagamento: fornitore_servizi_bancari --> cliente +
( (invio_quota_pagamento: fornitore_servizi_bancari  --> compagnia_aerea ;
invio_biglietto: compagnia_aerea --> cliente) | 
invio_quota_pagamento: fornitore_servizi_bancari --> acmesky ;
1 + richiesta_distanza_domicilio_aeroporto: acmesky --> maps ;
1 + (proposta_trasferimento: acmesky --> cliente ; 
1 + (richiesta_distanze_domicilio_compagnie_trasporto: acmesky --> maps ;
prenotazione_trasferimento: acmesky --> compagnia_trasporto_vicina))))))
```
Per illustrare ogni punto di questa coreografia, l'articoliamo nei 4 blocchi seguenti.

<!--
title: "Articolazione della coreografia in sotto-coreografie"
-->
```javascript
C::= registrazione_interesse_utente | richiesta_e_inoltro_offerte |
ricezione_e_inoltro_offerte_LM | acquisto_offerta
```

Passiamo in rassegna ciascun blocco, esplicitando la semantica di ogni comunicazione, nonché le condizioni di connectedness rispettate.

### Registrazione interessi utente

<!--
title: "Sotto-coreografia: registrazione_interessi_utente"
-->
```javascript
registrazione_interesse_utente ::= (invio_interesse: cliente --> acmesky)*
```

Nell'ambito della comunicazione `invio_interesse`, il cliente esprime le sue esigenze relativamente al biglietto aereo di andata ed eventualmente di ritorno che vuole acquistare: in dettaglio, specifica il periodo in cui intende viaggiare (quindi finestra temporale per l'andata ed eventuale finestra per il ritorno) e il prezzo massimo che è disposto a pagare. 

Abbiamo contrassegnato la comunicazione con `*` affinché lo stesso cliente possa esprimere più di un interesse, in momenti differenti.

### Richiesta e inoltro offerte

<!--
title: "Sotto-coreografia: richiesta_e_inoltro_offerte"
-->
```javascript
richiesta_e_inoltro_offerte ::= (
  ((richiesta_offerte: acmesky --> compagnia_aerea_a ;
risposta_offerte: compagnia_aerea_a --> acmesky) | ... |
(richiesta_offerte: acmesky --> compagnia_aerea_z ; 
risposta_offerte: compagnia_aerea_z --> acmesky)) ; 
1 + (invio_codici: acmesky --> prontogram ;
(inoltro_codice: prontogram --> cliente_1 ; ... ; 
inoltro_codice: prontogram --> cliente_n))
```

ACMESky, quotidianamente, ripete la comunicazione `richiesta_offerte`, entro cui chiede ad una singola compagnia aerea che le restituisca tutte le offerte attive. A questa comunicazione, ne segue un'altra, `risposta_offerte`, con cui la compagnia aerea contattata invia ad ACMESky quanto richiestole.

<!-- theme: warning -->
> Abbiamo adottato la notazione `...` al fine di catturare il fatto che le due comunicazioni appena descritte sono eseguite parallelamente per ogni compagnia aerea convenzionata con ACMESky.

Dopo che la/le compagnia/e aerea/e hanno risposto, ACMESky verifica la presenza di corrispondenze tra le offerte che gli sono giunte e gli interessi dei suoi clienti che ha memorizzato fino a quel momento:
- in caso affermativo, per ogni corrispondenza, Acmesky genera un codice che identifica la coppia offerte e il cliente potenzialmente interessato e lo/li al servizio di messaggistica Prontogram con la comunicazione `invio_codici`; Prontogram, con la comunicazione `inoltro_codice`, inoltrerà, in maniera sequenziale, i codici ai clienti corrispondenti; 
  - abbiamo adottato la notazione `; ... ;` per esplicitare il fatto che i vari inoltri di codici da Prontogram ai clienti interessati avvengono in maniera sequenziale;
- in caso negativo, non segue alcuna comunicazione.

<!-- theme: danger -->
> in dettaglio, per ogni offerta, ACMESky invia a Prontogram una *descrizione*, i dati identificativi del/dei destinatari e tanti *codici* quanti sono i clienti destinatari: ogni codice individua univocamente la coppia offerta-cliente; se individuasse univocamente solo l'offerta, un qualsiasi cliente che ne ha ricevuto uno potrebbe diffonderlo, permettendo così anche a soggetti terzi di usufruire delle offerte: noi vogliamo che ACMESky abbia il pieno controllo sulle offerte inviate alla sua clientela, per cui vogliamo che un utente qualsiasi possa ottenere un codice valido solo per sè e solo per mezzo di ACMESky stesso. Un vantaggio di questa soluzione potrebbe essere la possibilità per ACMESky di condurre analisi di mercato, conoscendo esattamente i clienti a cui ha inviato le offerte e quanti hanno poi finalizzato l'acquisto.

### Ricezione e inoltro offerte LM

<!--
title: "Sotto-coreografia: ricezione_e_inoltro_offerte_LM"
-->
```javascript
ricezione_e_inoltro_offerte_LM ::= (
  (invio_offerte_LM: compagnia_aerea_a --> acmesky ;
1 + (invio_codici_LM: acmesky --> prontogram ; 
(inoltro_codice_LM: prontogram --> cliente_1a ; ... ;
inoltro_codice_LM: prontogram --> cliente_na))) | 
... |
(invio_offerte_LM: compagnia_aerea_z --> acmesky ;
1 + (invio_codici_LM: acmesky --> prontogram ; 
(inoltro_codice_LM: prontogram --> cliente_1z ; ... ; 
inoltro_codice_LM: prontogram --> cliente_nz)) ) )
```

Nell'ambito della comunicazione `invio_offerte_LM`, una certa compagnia aerea invia ad ACMESky una o più offerte last-minute che ha attivato. Quindi, ACMESky verifica nuovamente la presenza di una corrispondenza tra il nuovo insieme di offerte memorizzate e gli interessi specificate dai suoi clienti fino a quel momento; a seconda che la verifica abbia esito positivo o meno, si eseguono le stesse comunicazioni di invio e inoltro delle offerte viste nella sezione precedente.

Sebbene trattasi delle stesse comunicazioni, abbiamo deciso di chiamarle diversamente (`invio_codici_LM` e `inoltro_codice_LM`), al fine di evitare problemi di concorrenza: dato che la 2° e 4° sotto-coreografia sono in esecuzione parallela, nello stesso istante potrebbero essere eseguite le stesse comunicazioni tra gli stessi partecipanti, mentre cambiandone il nome non si pone il problema.

<!-- theme: warning -->
> Abbiamo aggiunto un'altra volta la notazione `...` per catturare il fatto che ciascuna compagnia aerea convenzionata possa inviare offerte LM in qualsiasi momento.


### Acquisto biglietto
<!--
title: "Sotto-coreografia: acquisto_offerta"
-->
```javascript
acquisto_offerta ::= invio_codice_offerta: cliente --> acmesky ;
(notifica_errore_codice: acmesky --> cliente) +
(richiesta_link_pagamento: acmesky --> fornitore_servizi_bancari ;
invio_link_pagamento: fornitore_servizi_bancari --> acmesky;
inoltro_link_pagamento: acmesky --> cliente ;
pagamento: cliente --> fornitore_servizi_bancari ;
(notifica_errore_pagamento: fornitore_servizi_bancari --> cliente +
( (invio_quota_pagamento: fornitore_servizi_bancari  --> compagnia_aerea ;
invio_biglietto: compagnia_aerea --> cliente) | 
invio_quota_pagamento: fornitore_servizi_bancari --> acmesky ;
1 + richiesta_distanza_domicilio_aeroporto: acmesky --> maps ;
1 + (proposta_trasferimento: acmesky --> cliente ; 
1 + (richiesta_distanze_domicilio_compagnie_trasporto: acmesky --> maps ;
prenotazione_trasferimento: acmesky --> compagnia_trasporto_vicina)))))
```
Nell'ambito della comunicazione `invio_codice_offerta`, il cliente invia ad ACMESky il codice che identifica sè e l'offerta che intende acquistare; ACMESky verifica la correttezza del codice ricevuto:
- in caso sia errato, ACMESky lo notifica al cliente;
- in caso sia corretto, mediante le comunicazioni `richiesta_link_pagamento` e `invio_link_pagamento`, Acmesky richiede e ottiene dal fornitore dei servizi bancari il link di pagamento che quindi inoltra con `inoltro_link_pagamento` al cliente; il cliente apre il link e, con la comunicazione `pagamento` realizza la transazione con il fornitore dei servizi bancari;
  - in caso il pagamento sia fallito, il fornitore dei servizi bancari lo notifica al cliente;
  - in caso il pagamento abbia avuto successo, il fornitore dei servizi bancari, in parallelo, invia la quota dovuta alla compagnia aerea e ad ACMESky, con la comunicazione `invio_quota_pagamento`;
    - dopo che la compagnia aerea ha ricevuto la sua quota, con la comunicazione `invio_biglietto`, invia il biglietto dell'offerta acquistata al cliente;
    - non appena ACMESky ha ricevuto la sua quota, verifica che sussistano le condizioni per proporre al cliente il servizio di trasporto (mediante `richiesta_distanza_domicilio_aeroporto`) e, in caso positivo, invia la proposta al cliente con la comunicazione `proposta_trasferimento`: nel caso l'utente rifiuti, non segue alcuna comunicazione, diversamente ACMESky con la comunicazione `richiesta_distanze_domicilio_compagnie_trasporto` richiede le distanze tra il domicilio del cliente e le sedi delle compagnie di trasporti e con `prenotazione_trasferimento` effettua la prenotazione con la compagnia di trasporto più vicina al domicilio del cliente. 


## Verifica delle condizioni di connectedness
In prima battuta, abbiamo dovuto comprendere se lo scenario in oggetto rientrasse nel caso sincrono o in quello asincrono; a dirimere la questione sono stati, tra gli altri:
- i *clienti* i quali possono indicare le proprie esigenze sul portale di ACMESky più volte, ancor prima di ricevere le eventuali offerte corrispondenti;
- le offerte last-minute che possono essere inviate in qualsiasi momento dalle compagnie aeree ad ACMESky.

Pertanto, siamo giunti alla conclusione che questo scenario rientri nel caso **asincrono**.

Quindi, abbiamo valutato ogni operatore sequenziale (`;`), ciascuna scelta non deterministica (` + `), ciascuna iterazione (`*`), nonchè l’utilizzo multiplo di medesime operazioni: quindi, abbiamo rifinito la coreografia scritta in principio, migliorando le proprietà di connectedness, arrivando alla coreografia sopra presentata, che risulta essere connessa per le scelte condizionali e per l'operatore sequenza secondo il pattern **Sender**.

Per quanto riguarda la connessione per la sequenza, adottiamo la seguente formalizzazione per riferirci ai partecipanti in ruolo di mittente e destinatario delle operazioni consecutive:

```javascript
operazione_1: a --> b; operazione2: c --> d
```

Per una maggiore chiarezza, riproponiamo l'articolazione della coregrafia di cui sopra.
<!--
title: "Articolazione della coreografia in sotto-coreografie"
-->
```javascript
C::= registrazione_interesse_utente | richiesta_e_inoltro_offerte |
ricezione_e_inoltro_offerte_LM | acquisto_offerta
```

Le diverse sotto-coreografie sono eseguite in parallelo: ai fini della connectedness, non vi sono condizioni per la composizione parallela, per cui possiamo ridurre la nostra analisi direttamente al livello delle sotto-coreografie. 

### Registrazione interessi utente

<!--
title: "Sotto-coreografia: registrazione_interesse_utente"
-->
```javascript
registrazione_interesse_utente ::= (invio_interesse: cliente --> acmesky)*
```

Dato che la comunicazione `invio_interesse` è contrassegnata dall'operatore di iterazione (`*`) dobbiamo verificare la connessione tanto sulla sequenza quanto sulla scelta non deterministica tra il ripetere l'operazione stessa e procedere nella coreografia:
- in caso l'operazione sia ripetuta, si ha connessione per sequenza in quanto **a = c**;
- in caso l'operazione non sia ripetuta, la coreografia non presenta comunicazioni successive, per cui sono rispettate tutte le condizioni di connessione.

Si ha connessione anche per scelta dal momento che, in caso non si ripeta, non segue alcuna comunicazione.

### Richiesta e inoltro offerte

<!--
title: "Sotto-coreografia: richiesta_e_inoltro_offerte"
-->
```javascript
richiesta_e_inoltro_offerte ::= (
  ((richiesta_offerte: acmesky --> compagnia_aerea_a ;
risposta_offerte: compagnia_aerea_a --> acmesky) | ... |
(richiesta_offerte: acmesky --> compagnia_aerea_z ; 
risposta_offerte: compagnia_aerea_z --> acmesky)) ; 
1 + (invio_codici: acmesky --> prontogram ;
(inoltro_codice: prontogram --> cliente_1 ; ... ; 
inoltro_codice: prontogram --> cliente_n)))
```

La comunicazione `richiesta_offerte` è connessa per sequenza con l'operazione seguente (`risposta_offerte`) in quanto **b = c**.

La comunicazione `risposta_offerte` è connessa per la sequenza con `invio_codice` in quanto **b = c** e con la comunicazione nulla (`1`). Vi è connessione per la scelta seguente, essendo uno dei due branch 1.

La comunicazione `invio_codici` è connessa per la sequenza con `inoltro_codice`, in quanto **b = c**. 

La comunicazione `inoltro_codice` è connessa per la sequenza con `inoltro_codice` consecutiva, in quanto **a = c**. 

### Ricezione e inoltro offerte LM

<!--
title: "Sotto-coreografia: ricezione_e_inoltro_offerte_LM"
-->
```javascript
ricezione_e_inoltro_offerte_LM ::= (
  invio_offerte_LM: compagnia_aerea_a --> acmesky ;
1 + (invio_codici_LM: acmesky --> prontogram ; 
(inoltro_codice_LM: prontogram --> cliente_1a ; ... ;
inoltro_codice_LM: prontogram --> cliente_na))) | 
... |
(invio_offerte_LM: compagnia_aerea_z --> acmesky ;
1 + (invio_codici_LM: acmesky --> prontogram ; 
(inoltro_codice_LM: prontogram --> cliente_1z ; ... ; 
inoltro_codice_LM: prontogram --> cliente_nz)) )
```

La comunicazione `invio_offerte_LM` è connessa per sequenza con `invio_codici_LM` in quanto **b = c** e con `1`. La scelta seguente è connessa, essendo uno dei due branch `1`.

La comunicazione `invio_codici_LM` è connessa per sequenza con l'operazione seguente (`inoltro_codice_LM`) in quanto **b = c**.

La comunicazione `inoltro_codice_LM` è connessa per la sequenza con `inoltro_codice_LM` consecutiva, in quanto **a = c**. 

### Acquisto biglietto
<!--
title: "Sotto-coreografia: acquisto_offerta"
-->
```javascript
acquisto_offerta ::= invio_codice_offerta: cliente --> acmesky ;
(notifica_errore_codice: acmesky --> cliente) +
(richiesta_link_pagamento: acmesky --> fornitore_servizi_bancari ;
invio_link_pagamento: fornitore_servizi_bancari --> acmesky;
inoltro_link_pagamento: acmesky --> cliente ;
pagamento: cliente --> fornitore_servizi_bancari ;
(notifica_errore_pagamento: fornitore_servizi_bancari --> cliente +
( (invio_quota_pagamento: fornitore_servizi_bancari  --> compagnia_aerea ;
invio_biglietto: compagnia_aerea --> cliente) | 
invio_quota_pagamento: fornitore_servizi_bancari --> acmesky ;
1 + richiesta_distanza_domicilio_aeroporto: acmesky --> maps ;
1 + (proposta_trasferimento: acmesky --> cliente ; 
1 + (richiesta_distanze_domicilio_compagnie_trasporto: acmesky --> maps ;
prenotazione_trasferimento: acmesky --> compagnia_trasporto_vicina)))))
```
La comunicazione `invio_codice_offerta` è connessa per sequenza con i due branch seguenti, in quanto, in entrambi i casi, **b = c**. L'operatore di scelta successivo è connesso in quanto i due branch hanno in comune l'iniziatore.

La comunicazione `richiesta_link_pagamento` è connessa per sequenza con l'operazione seguente (`invio_link_pagamento`) in quanto **b = c**.

La comunicazione `invio_link_pagamento` è connessa per sequenza con l'operazione seguente (`inoltro_link_pagamento`) in quanto **b = c**.

La comunicazione `inoltro_link_pagamento` è connessa per sequenza con l'operazione seguente (`pagamento`) in quanto **b = c**.

La comunicazione `pagamento` è connessa per sequenza con i due branch seguenti (`notifica_errore_pagamento` e `invio_quota_pagamento`), in quanto, in entrambi i casi, **b = c**.

Le comunicazioni `notifica_errore_pagamento` e `invio_quota_pagamento` sono connesse per scelta in quanto hanno in comune l'iniziatore.

La comunicazione `invio_quota_pagamento` è connessa per sequenza con l'operazione seguente (`invio_biglietto`) in quanto **b = c**.

Seguitamente, la comunicazione `invio_quota_pagamento` è connessa per sequenza tanto con `1` quanto con `richiesta_distanza_domicilio_aeroporto` in quanto **b = c**.

La comunicazione `richiesta_distanza_domicilio_aeroporto` è connessa per sequenza tanto con `1` quanto con `proposta_trasferimento` in quanto **a = c**.

La comunicazione `proposta_trasferimento` è connessa per sequenza tanto con `1` quanto con `proposta_trasferimento` in quanto **a = c**.

Ancora, la comunicazione `proposta_trasferimento` è connessa per sequenza tanto con `1` quanto con `richiesta_distanze_domicilio_compagnie_trasporto` in quanto **a = c**

Gli ultimi tre operatori di scelta sono connessi, essendo uno dei due branch `1`.

La comunicazione `richiesta_distanze_domicilio_compagnie_trasporto` è connessa per sequenza con `prenotazione_trasferimento` in quanto **a = c**.

Pertanto, possiamo concludere che l'intera coreografia risulta essere connessa secondo il pattern **Sender**.

## Proiezioni
Una volta verificata la connessione della coreografia, abbiamo proceduto col proiettarla in un sistema di ruoli.
Di seguito, riportiamo la proiezione per ogni partecipante.

<!-- theme: warning -->
> La notazione convenzionale prevede che le operazioni di output su un certo ruolo siano sopralineati: a causa dei vincoli alla formattazione posti dall'engine markdown di *Stoplight*, sostituiamo la sopralineatura col carattere `^` che precede l'operazione di output.

<!--
title: "Proiezione su cliente"
-->
```javascript
proj(C, cliente): (^invio_interesse@acmesky)* | (((1 ; 1) | ... |
(1 ; 1)) ; 1 + (1 ; (1 ; ... ; 1))) | ((1 ; 1 + (1 ; (1 ; ... ; 1)))
| ... | (1 ; 1 + (1 ; (1 ; ... ; 1)) )) | (^invio_codice_offerta@acmesky ;
(notifica_errore_codice@acmesky) +
(1 ; 1 ; inoltro_link_pagamento@acmesky ; ^pagamento@fornitore_servizi_bancari ;
(notifica_errore_pagamento@fornitore_servizi_bancari +
( (1 ; invio_biglietto@compagnia_aerea) | 1 ; 1 + 1 ; 1 + 
(proposta_trasferimento@acmesky ; 1 + (1 ; 1))))))


```
---
<!--
title: "Proiezione su cliente_1"
-->
```javascript
proj(C, cliente_1): (1)* | (((1 ; 1) | ... | (1 ; 1)) ; 1 + (1 ;
(inoltro_codice@prontogram ; ... ; 1))) | ((1 ; 1 + (1 ; (1 ; ... ; 1)))
| ... | (1 ; 1 + (1 ; (1 ; ... ; 1)) )) | (1 ; 1) + (1 ; 1 ; 1 ; 1 ;
(1 + ((1 ; 1) |  1 ; 1 + 1 ; 1 + (1 ; 1 + (1 ; 1)))))
```
---
<!--
title: "Proiezione su cliente_n"
-->
```javascript
proj(C, cliente_n):  (1)* | (((1 ; 1) | ... | (1 ; 1)) ; 1 + (1 ;
(1 ; ... ; inoltro_codice@prontogram))) | ((1 ;
1 + (1 ; (1 ; ... ; 1))) | ... | (1 ; 1 + (1 ; (1 ; ... ; 1)) )) |
(1 ; 1) + (1 ; 1 ; 1 ; 1 ; (1 + ((1 ; 1) | 
1 ; 1 + 1 ; 1 + (1 ; 1 + (1 ; 1)))))
```
---
<!--
title: "Proiezione su cliente_1a"
-->
```javascript
proj(C, cliente_1a): (1)* | (((1 ; 1) | ... | (1 ; 1)) ;
1 + (1 ; (1 ; ... ; 1))) | ((1 ;
1 + (1 ; (inoltro_codice_LM@prontogram ; ... ; 1))) | ... | (1 ; 1 +
(1 ; (1 ; ... ; 1)) )) |(1 ; 1) + (1 ; 1 ; 1 ; 1 ; (1 + ((1 ; 1) | 
1 ; 1 + 1 ; 1 + (1 ; 1 + (1 ; 1)))))
```
---
<!--
title: "Proiezione su cliente_1z"
-->
```javascript
proj(C, cliente_1z): (1)* | (((1 ; 1) | ... | (1 ; 1)) ; 1 +
(1 ; (1 ; ... ; 1))) | ((1 ; 1 + (1 ; (1 ; ... ; 1))) | ... |
(1 ; 1 + (1 ; (inoltro_codice_LM@prontogram ; ... ; 1)) )) |
(1 ; 1) + (1 ; 1 ; 1 ; 1 ; (1 + ((1 ; 1) | 
1 ; 1 + 1 ; 1 + (1 ; 1 + (1 ; 1)))))
```
---
<!--
title: "Proiezione su cliente_na"
-->
```javascript
proj(C, cliente_na): (1)* | (((1 ; 1) | ... | (1 ; 1)) ;
1 + (1 ; (1 ; ... ; 1))) | ((1 ; 1 + (1 ; (1 ; ... ; inoltro_codice_LM@prontogram)))
| ... | (1 ; 1 + (1 ; (1 ; ... ; 1)) )) |
(1 ; 1) + (1 ; 1 ; 1 ; 1 ; (1 + ((1 ; 1) | 
1 ; 1 + 1 ; 1 + (1 ; 1 + (1 ; 1)))))
```
---
<!--
title: "Proiezione su cliente_nz"
-->
```javascript
proj(C, cliente_nz): (1)* | (((1 ; 1) | ... | (1 ; 1)) ; 1 +
(1 ; (1 ; ... ; 1))) | ((1 ; 1 + (1 ; (1 ; ... ; 1))) | ... |
(1 ; 1 + (1 ; (1 ; ... ; inoltro_codice_LM@prontogram)) )) |
(1 ; 1) + (1 ; 1 ; 1 ; 1 ; (1 + ((1 ; 1) | 
1 ; 1 + 1 ; 1 + (1 ; 1 + (1 ; 1)))))
```
---
<!--
title: "Proiezione su compagnia_trasporto_vicina"
-->
```javascript
proj(C, compagnia_trasporto_vicina): (1)* | (((1 ; 1) | ... |
(1 ; 1)) ; 1 + (1 ; (1 ; ... ; 1))) | ((1 ;
1 + (1 ; (1 ; ... ; 1))) | ... | (1 ; 1 + (1 ; (1 ; ... ; 1)) )) |
(1 ; 1) + (1 ; 1 ; 1 ; 1 ; (1 + ((1 ; 1) | 
1 ; 1 + 1 ; 1 + (1 ; 1 + (1 ; prenotazione_trasferimento@acmesky)))))
```
---
<!--
title: "Proiezione su fornitori_serivizi_bancari"
-->
```javascript
proj(C, fornitori_serivizi_bancari): (1)* | (((1 ; 1) | ... |
(1 ; 1)) ; 1 + (1 ; (1 ; ... ; 1))) | ((1 ; 1 + (1 ; (1 ; ... ; 1)))
| ... | (1 ; 1 + (1 ; (1 ; ... ; inoltro_codice_LM@prontogram)) )) |
(1 ; (1) + (richiesta_link_pagamento@acmesky ;
^invio_link_pagamento@acmesky ; 1 ; pagamento@cliente ;
(^notifica_errore_pagamento@cliente + ( (^invio_quota_pagamento@compagnia_aerea ;
1) | ^invio_quota_pagamento@acmesky ; 1 + 1 ; 1 + (1 ; 1 + (1 ; 1))))))
```
---
<!--
title: "Proiezione su compagnie_aerea_a"
-->
```javascript
proj(C, compagnie_aerea_a): 

(1)* | (( (richiesta_offerte@acmesky ; ^risposta_offerte@acmesky)
| ... | (1;1) ) ; 1 + (1 ; 1 | ... |1 )) |
((^last_minute@acmesky ; 1 + (1; 1 | ... | 1)) | ... |
(1 ; 1 + (1 ; 1 | ... | 1 )) ) | (1 ; 1 + (1 ; 1 ; 1 + ((1 ; 1 ) |
1 ; 1 + (1 ; 1 + 1))) )
```
---
<!--
title: "Proiezione su compagnia_aerea_z"
-->
```javascript
proj(C, compagnia_aerea_z): (1)* | (((richiesta_offerte@acmesky ;
^risposta_offerte@acmesky) | ... | (1 ; 1)) ; 1 + (1 ; (1 ; ... ; 1))) |
((^invio_offerte_LM@acmesky ;
1 + (1 ; (1 ; ... ; 1))) | ... | (1 ; 1 + (1 ; (1 ; ... ; 1)) )) |
(1 ; 1) + (1 ; 1 ; 1 ; 1 ; (1 + ((1 ; 1) | 
1 ; 1 + 1 ; 1 + (1 ; 1 + (1 ; 1)))))
```
---
<!--
title: "Proiezione su acmesky"
-->
```javascript
proj(C, acmesky): (invio_interesse@cliente)* | (((^richiesta_offerte@compagnia_aerea_a ;
risposta_offerte@compagnia_aerea_a) | ... |
(^richiesta_offerte@compagnia_aerea_z ; 
risposta_offerte@acmesky)) ; 
1 + (^invio_codici@prontogram ;
(1 ; ... ; 1))) | ((invio_offerte_LM@compagnia_aerea_a ;
1 + (^invio_codici_LM@prontogram ; 
(1 ; ... ; 1))) | ... | (invio_offerte_LM@compagnia_aerea_z ;
1 + (^invio_codici_LM@prontogram ; 
(1 ; ... ;1 )) )) | (invio_codice_offerta@cliente ;
(^notifica_errore_codice@acmesky) +
(^richiesta_link_pagamento@fornitore_servizi_bancari ;
invio_link_pagamento@fornitore_servizi_bancari ;
^inoltro_link_pagamento@cliente ;
1 ; (1 + ((1 ; 1) | 
invio_quota_pagamento@fornitore_servizi_bancari ;
1 + ^richiesta_distanza_domicilio_aeroporto@maps ;
1 + (^proposta_trasferimento@cliente ; 
1 + (^richiesta_distanze_domicilio_compagnie_trasporto@maps ;
^prenotazione_trasferimento@compagnia_trasporto_vicina))))))
```
---
<!--
title: "Proiezione su prontogram"
-->
```javascript
proj(C, prontogram): (1)* | 
(((1 ; 1) | ... | (1 ; 1)) ; 1 + (invio_codici@acmesky ;
(^inoltro_codice@cliente_1 ; ... ; 
^inoltro_codice@cliente_n))) | ((1 ;
1 + (invio_codici_LM@acmesky ; 
(^inoltro_codice_LM@cliente_1a ; ... ;
^inoltro_codice_LM@cliente_na))) | 
... | (1 ; 1 + (invio_codici_LM@acmesky ; 
(^inoltro_codice_LM@cliente_1z ; ... ; 
^inoltro_codice_LM@cliente_nz)) )) | (1 ;
(1) + (1 ; 1 ; 1 ; 1 ; (1 + ( (1 ; 1) | 1 ;
1 + 1 ; 1 + (1 ; 1 + (1 ; 1))))))
```

