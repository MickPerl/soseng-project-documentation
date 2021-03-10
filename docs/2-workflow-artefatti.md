# Workflow e artefatti
Di seguito, riportiamo le fasi salienti per la realizzazione del progetto, articolate in due parti: siamo partiti col modellizzare il problema su diversi livelli e da differenti prospettive, per poi procedere con l'implementazione della soluzione applicativa progettata.

## Modellazione

### 1. Modellazione delle comunicazioni
La prima attività che abbiamo compiuto è stata la lettura attenta della descrizione del dominio e del problema.<br>
Una volta che abbiamo individuato i partecipanti coinvolti e le operazioni svolte, abbiamo proceduto col modellare le comunicazioni dello scenario usando una **coreografia**: per la sua definizione formale ci siamo serviti di un *process calculi*, essendo più accurato di WS-CDL e della formalizzazione per le coreografie di BPMN:  mediante il suo formalismo, abbiamo potuto comprendere in maniera puntuale quanto accade nello scenario, nonchè raffinare la coreografia stessa per migliorarne le *proprietà di connectedness*. Queste ultime sappiamo essere proprietà sufficienti (e non necessarie) a garantire che la successiva **proiezione in un sistema di ruoli** sia corretta .<br>
[Link alla coreografia e alle proiezioni](3-coreografia.md "Clicca per andare agli artefatti corrispondenti")

### 2. Modellazione dei processi
Abbiamo modellato l'intera realtà della piattaforma ACMESky, compresi i dettagli dei partecipanti di cui integra i servizi, in un **diagramma BPMN**: tale modellazione ha scopo documentativo, il che giustifica il suo livello di dettaglio (alcuni partecipanti esterni ad ACMESky sono presentati come *collapsed pool*).
[Link al diagramma BPMN](4-diagramma-BPMN.md "Clicca per andare all'artefatto corrispondente")

### 3. Progettazione di una SOA e documentazione in UML
Abbiamo progettato una *Service Oriented Architecture* (SOA) che realizzi la piattaforma in considerazione, che abbiamo quindi documentato utilizzando UML: in dettaglio, ci siamo serviti del profilo *TinySOA*, in quanto mette a disposizione gli artefatti e gli stereotipi appropriati per le nostre esigenze documentative.

[Link alla modello UML](5-modello-UML.md "Clicca per andare all'artefatto corrispondente")

### 5. Realizzazione della piattaforma
Abbiamo realizzato il sistema, implementando la logica dei partecipanti in **NodeJS** e **Jolie** e adottando, come Business Process Management System (BPMS), quello offerto da **Camunda** (Wildfly).

[Link alle scelte implementative](5-modello-UML.md "Clicca per andare agli artefatti corrispondenti")
