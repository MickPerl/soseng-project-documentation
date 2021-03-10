# Coreografia e proiezioni

Abbiamo modellato le comunicazioni dello scenario in oggetto definendo una coreografia: in dettaglio, per la sua definizione formale ci siamo serviti di un *process calculi*, essendo più accurato di WS-CDL e della formalizzazione per le coreografie di BPMN.

Abbiamo poi verificato le proprietà di connectedness e quindi proiettato l'output in un sistema di ruoli.

## Coreografia
La coreografia cui siamo giunti dopo numerosi confronti, riflessioni e iterazioni è presentata di seguito: 

```javascript
C ::= (invio_interessi: cliente --> acmesky)*

|

(( (richiesta_offerte: acmesky --> compagnia_aerea_a ; risposta_offerte: compagnia_aerea_a --> acmesky) |   
... |
(richiesta_offerte: acmesky --> compagnia_aerea_z ;
risposta_offerte: compagnia_aerea_z --> acmesky) ) ; 
1 + (invia_offerte: acmesky --> prontogram ;
inoltra_offerte: prontogram --> cliente_1 |
... |
inoltra_offerte: prontogram --> cliente_n ))

|     

((last_minute: compagnia_aerea_a --> acmesky ; 1 +               (invia_offerte_LM: acmesky --> prontogram ;
inoltra_offerte_LM: prontogram --> cliente_1a | ... | inoltra_offerte_LM: prontogram --> cliente_na)) | 
... |
(last_minute: compagnia_aerea_z --> acmesky ; 1 + 
(invia_offerte_LM: acmesky --> prontogram ; 
inoltra_offerte_LM: prontogram --> cliente_1z | ... | inoltra_offerte_LM: prontogram --> cliente_nz ))
)

| 

(invio_codice_offerta: cliente --> acmesky ;
1 + (inoltro_servizi_bancari: acmesky --> cliente ;
pagamento: cliente --> fornitore_servizi_bancari ;
1 + ((invio_pagamento: fornitore_servizi_bancari  --> compagnia_aerea ;
invio_biglietto: compagnia_aerea --> cliente ) | 
invio_pagamento: fornitore_servizi_bancari --> acmesky ;
1 + (propongo_trasferimento: acmesky --> cliente ; 
1 + prenota_trasferimento: acmesky --> compagnia_vicina))))
```