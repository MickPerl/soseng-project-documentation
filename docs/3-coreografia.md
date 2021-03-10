# Coreografia e proiezioni

Abbiamo modellato le comunicazioni dello scenario in oggetto definendo una coreografia: in dettaglio, per la sua definizione formale ci siamo serviti di un *process calculi*, essendo più accurato di WS-CDL e della formalizzazione per le coreografie di BPMN.

Abbiamo poi verificato le proprietà di connectedness e quindi proiettato l'output in un sistema di ruoli.

## Coreografia
La coreografia cui siamo giunti dopo numerosi confronti, riflessioni e iterazioni è presentata di seguito: 

```javascript
C ::= (invio_interessi: cliente --> acmesky)*

|

(( (richiesta_offerte: acmesky --> compagnia_aerea_a ;
risposta_offerte: compagnia_aerea_a --> acmesky) |   
... |
(richiesta_offerte: acmesky --> compagnia_aerea_z ;
risposta_offerte: compagnia_aerea_z --> acmesky) ) ; 
1 + (invio_offerte: acmesky --> prontogram ;
inoltro_offerte: prontogram --> cliente_1 |
... |
inoltro_offerte: prontogram --> cliente_n ))

|     

( (last_minute: compagnia_aerea_a --> acmesky ;
1 + (invio_offerte_LM: acmesky --> prontogram ;
inoltro_offerte_LM: prontogram --> cliente_1a |
... |
inoltro_offerte_LM: prontogram --> cliente_na) ) | 
... |
(last_minute: compagnia_aerea_z --> acmesky ;
1 + (invio_offerte_LM: acmesky --> prontogram ; 
inoltro_offerte_LM: prontogram --> cliente_1z |
... |
inoltro_offerte_LM: prontogram --> cliente_nz) ))

| 

(invio_codice_offerta: cliente --> acmesky ;
1 + (invia_link_banca: acmesky --> cliente ;
pagamento: cliente --> fornitore_servizi_bancari ;
1 + ((invio_quota_pagamento: fornitore_servizi_bancari  --> compagnia_aerea ;
invio_biglietto: compagnia_aerea --> cliente ) | 
invio_quota_pagamento: fornitore_servizi_bancari --> acmesky ;
1 + (proposta_trasferimento: acmesky --> cliente ; 
1 + prenotazione_trasferimento: acmesky --> compagnia_trasporto_vicina) )))
```

Per illustrare ogni punto di questa coreografia, l'articoliamo nei 4 blocchi seguenti.

```javascript
C::= registrazione_interessi_utente | richiesta_e_inoltro_offerte | ricezione_e_inoltro_offerte_LM | acquisto_biglietto
```

Passiamo in rassegna ciascun blocco, esplicitando la semantica di ogni comunicazione, nonché le proprietà di connectedness soddisfatte.

### Registrazione interessi utente

<!--
title: "Parte della coreografia: registrazione_interessi_utente"
-->
```javascript
registrazione_interessi_utente ::= (invio_interessi: cliente --> acmesky)*
```

Nell'ambito della comunicazione `invio_interesse`, il cliente esprime le sue esigenze relativamente al biglietto aereo di andata e ritorno che intede acquistare: in dettaglio, specifica il periodo il periodo in cui intende viaggiare e il prezzo massimo che è disposto a pagare. 

Abbiamo contrassegnato la comunicazione con `*` affinché lo stesso cliente possa esprimere più volte le sue esigenze: dal suo canto, ACMESky, nel ricercare le offerte, considererà per ogni utente sempre e solo la sua ultima specificazione.

### Richiesta e inoltro offerte

<!--
title: "Parte della coreografia: richiesta_e_inoltro_offerte"
-->
```javascript
richiesta_e_inoltro_offerte ::= ( (richiesta_offerte: acmesky --> compagnia_aerea_a ;
risposta_offerte: compagnia_aerea_a --> acmesky) |   
... |
(richiesta_offerte: acmesky --> compagnia_aerea_z ;
risposta_offerte: compagnia_aerea_z --> acmesky) ) ; 
1 + (invio_offerte: acmesky --> prontogram ;
inoltro_offerte: prontogram --> cliente_1 |
... |
inoltro_offerte: prontogram --> cliente_n )
```

ACMESky, quotidianamente, ripete la comunicazione `richiesta_offerte`, entro cui chiede ad una singola compagnia aerea che le restituisca tutte le offerte attive. A questa comunicazione, ne segue un'altra, `risposta_offerte`, con cui la compagnia aerea contattata invia ad ACMESky quanto richiestole.

<!-- theme: warning -->
> Abbiamo adottato la notazione `...` al fine di catturare il fatto che le due comunicazioni appena descritte sono eseguite parallelamente per ogni compagnia aerea convenzionata con ACMESky.

Dopo che la/le compagnia/e aerea/e hanno risposto, ACMESky verifica la presenza di una corrispondenza tra le offerte che gli sono giunte e le esigenze specificate dai suoi clienti fino a quel momento:
- in caso affermativo, invia la/le offerta/e (in dettaglio, per ogni offerta una descrizione, un codice identificativo e il/i cliente/i cui è indirizzata) al servizio di messagistica Prontogram con la comunicazione `invio_offerte`; Prontogram, con la comunicazione `inoltro_offerte`, inoltrerà questa/e offerta/e al destinatario indicatogli;
- in caso negativo, non segue alcuna comunicazione.

<!-- theme: warning -->
> Abbiamo nuovamente adottato la notazione `...` per catturare il fatto che Prontogram eseguirà in parallelo l'operazione `inoltro_offerte` per ciascun cliente che gli è stato indicato nelle comunicazioni `invio_offerte`.

### Ricezione e inoltro offerte LM

<!--
title: "Parte della coreografia: ricezione_e_inoltro_offerte_LM"
-->
```javascript
ricezione_e_inoltro_offerte_LM ::= (last_minute: compagnia_aerea_a --> acmesky ;
1 + (invio_offerte_LM: acmesky --> prontogram ;
inoltro_offerte_LM: prontogram --> cliente_1a |
... |
inoltro_offerte_LM: prontogram --> cliente_na) ) | 
... |
(last_minute: compagnia_aerea_z --> acmesky ;
1 + (invio_offerte_LM: acmesky --> prontogram ; 
inoltro_offerte_LM: prontogram --> cliente_1z |
... |
inoltro_offerte_LM: prontogram --> cliente_nz) )
```

Nell'ambito della comunicazione `last_minute`, una certa compagnia aerea invia ad ACMESky una sua offerta last-minute. Quindi, ACMESky verifica nuovamente la presenza di una corrispondenza tra il nuovo insieme di offerte memorizzate e le esigenze specificate dai suoi clienti fino a quel momento; a seconda che la verifica abbia esito positivo o meno, si eseguono le stesse comunicazioni di invio e inoltro delle offerte viste in [Richiesta e inoltro offerte](docs/3-coreografia.md#richiesta-e-inoltro-offerte).

Sebbene trattasi delle stesse comunicazioni, abbiamo deciso di chiamarle diversamente (`invio_offerte_LM` e `inoltro_offerte_LM`), al fine di evitare problemi di concorrenza: dato che la 2° e 4° sotto-coreografia sono in esecuzione parallela, nello stesso istante potrebbero essere eseguite le stesse comunicazioni tra gli stessi partecipanti, mentre cambiandone il nome non si pone il problema.

<!-- theme: warning -->
> Abbiamo aggiunto un'altra volta la notazione `...` per catturare il fatto che ciascuna compagnia aerea convenzionata possa inviare un'offerta LM in qualsiasi momento.


### Acquisto biglietto
<!--
title: "Parte della coreografia: acquisto_biglietto"
-->
```javascript
acquisto_biglietto ::= invio_codice_offerta: cliente --> acmesky ;
(notifica_errore_codice: acmesky --> cliente) +
(invia_link_banca: acmesky --> cliente ;
pagamento: cliente --> fornitore_servizi_bancari ;
(notifica_errore_pagamento: fornitore_servizi_bancari --> cliente) +
( (invio_quota_pagamento: fornitore_servizi_bancari  --> compagnia_aerea ;
invio_biglietto: compagnia_aerea --> cliente) | 
invio_quota_pagamento: fornitore_servizi_bancari --> acmesky ;
1 + (proposta_trasferimento: acmesky --> cliente ; 
1 + prenotazione_trasferimento: acmesky --> compagnia_trasporto_vicina) ))
```
Nell'ambito della comunicazione `invio_codice_offerta`, il cliente invia ad ACMESky il codice identificativo dell'offerta che intende acquistare; ACMESky verifica la correttezza del codice ricevuto:
- in caso sia scorretto, non segue alcuna comunicazione;
- in caso sia corretto, mediante la comunicazione `invia_link_banca`, ACMESky invia al cliente il link al fornitore di servizi bancari cui si appoggia per i pagamenti; il cliente apre il link e, con la comunicazione `pagamento` realizza la transazione con il fornitore dei servizi bancari;
  - in caso il pagamento sia fallito, non segue alcuna comunicazione;
  - in caso il pagamento abbia avuto successo, il fornitore dei servizi bancari, in parallelo, invia la quota dovuta alla compagnia aerea e ad ACMESky, con la comunicazione `invio_quota_pagamento`;
    - dopo che la compagnia aerea ha ricevuto la sua quota, con la comunicazione `invio_biglietto`, invia il biglietto dell'offerta acquistata al cliente;
    - non appena ACMESky ha ricevuto la sua quota, verifica che sussistano le condizioni per proporre al cliente il servizio di trasporto e, in caso positivo, invia la proposta al cliente con la comunicazione `proposta_trasferimento`: nel caso l'utente rifiuti, non segue alcuna comunicazione, diversamente ACMESky con la comunicazione `prenotazione_trasferimento` effettua la prenotazione con la compagnia di trasporto selezionata. 


## Proprietà di connectedness rispettate

### Registrazione interessi utente

<!--
title: "Parte della coreografia: registrazione_interessi_utente"
-->
```javascript
registrazione_interessi_utente ::= (invio_interessi: cliente --> acmesky)*
```

### Richiesta e inoltro offerte

<!--
title: "Parte della coreografia: richiesta_e_inoltro_offerte"
-->
```javascript
richiesta_e_inoltro_offerte ::= ( (richiesta_offerte: acmesky --> compagnia_aerea_a ;
risposta_offerte: compagnia_aerea_a --> acmesky) |   
... |
(richiesta_offerte: acmesky --> compagnia_aerea_z ;
risposta_offerte: compagnia_aerea_z --> acmesky) ) ; 
1 + (invio_offerte: acmesky --> prontogram ;
inoltro_offerte: prontogram --> cliente_1 |
... |
inoltro_offerte: prontogram --> cliente_n )
```

### Ricezione e inoltro offerte LM

<!--
title: "Parte della coreografia: ricezione_e_inoltro_offerte_LM"
-->
```javascript
ricezione_e_inoltro_offerte_LM ::= (last_minute: compagnia_aerea_a --> acmesky ;
1 + (invio_offerte_LM: acmesky --> prontogram ;
inoltro_offerte_LM: prontogram --> cliente_1a |
... |
inoltro_offerte_LM: prontogram --> cliente_na) ) | 
... |
(last_minute: compagnia_aerea_z --> acmesky ;
1 + (invio_offerte_LM: acmesky --> prontogram ; 
inoltro_offerte_LM: prontogram --> cliente_1z |
... |
inoltro_offerte_LM: prontogram --> cliente_nz) )
```



### Acquisto biglietto
<!--
title: "Parte della coreografia: acquisto_biglietto"
-->
```javascript
acquisto_biglietto ::= invio_codice_offerta: cliente --> acmesky ;
(notifica_errore_codice: acmesky --> cliente) +
(invia_link_banca: acmesky --> cliente ;
pagamento: cliente --> fornitore_servizi_bancari ;
(notifica_errore_pagamento: fornitore_servizi_bancari --> cliente) +
( (invio_quota_pagamento: fornitore_servizi_bancari  --> compagnia_aerea ;
invio_biglietto: compagnia_aerea --> cliente) | 
invio_quota_pagamento: fornitore_servizi_bancari --> acmesky ;
1 + (proposta_trasferimento: acmesky --> cliente ; 
1 + prenotazione_trasferimento: acmesky --> compagnia_trasporto_vicina) ))
```
