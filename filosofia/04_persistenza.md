# Arke — Persistenza

---

## La Persistenza come Dettaglio

Uno dei principi architetturali di Arke è che il core del sistema non sa — e non deve sapere — come i dati vengono fisicamente salvati.

La logica di dominio (cosa sono gli Arke, come si relazionano le Unit, come funzionano i cicli di vita) è completamente separata dal meccanismo di persistenza. Questo non è un dettaglio implementativo: è una scelta filosofica.

**Il dato ha un'identità indipendente dal posto in cui risiede.**

---

## Il Connettore: arke-postgres

Il ponte tra il core di Arke e il mondo esterno è il connettore. Nella configurazione standard, questo ruolo è svolto da **arke-postgres**.

arke-postgres implementa il contratto che il core si aspetta: sa come tradurre le operazioni del meta-modello (crea un Arke, trova tutte le Unit di un certo tipo, collega due entità) in operazioni concrete su PostgreSQL.

La separazione ha conseguenze pratiche importanti:

- Il core non contiene SQL, non conosce tabelle, non gestisce connessioni
- Il connettore non conosce la logica di dominio: riceve istruzioni e le esegue
- Sostituire o affiancare un connettore diverso non richiede di toccare il core

Il connettore è un adattatore, non un'estensione.

---

## La Memoria Veloce: ETS

Non tutti i dati hanno lo stesso profilo di accesso. Alcuni vengono letti raramente, altri vengono consultati ad ogni operazione.

La struttura degli Arke stessi — i tipi, i Parameters, le definizioni — è tra i dati più acceduti del sistema: ogni operazione sulle Unit deve verificare la conformità con il proprio tipo. Leggere queste informazioni da database ad ogni richiesta sarebbe un collo di bottiglia inutile.

Arke risolve questo attraverso **ETS** (Erlang Term Storage): una struttura di memoria condivisa, velocissima, disponibile nativamente nella piattaforma su cui Arke è costruito.

Le definizioni degli Arke, i Parameters, le configurazioni di sistema vivono in ETS. Sono "vicine" all'applicazione — in memoria, sullo stesso nodo, senza latenza di rete. Il database rimane la fonte di verità, ma non è il punto di accesso per ogni singola lettura.

Il risultato è un sistema che scala senza sacrificare la coerenza: i dati caldi sono sempre disponibili, i dati freddi vivono dove devono.

---

## Distribuito e Scalabile: libcluster

Un sistema che cresce deve poter girare su più nodi. Quando si scala orizzontalmente, la memoria locale non basta: lo stato condiviso deve essere coordinato.

Arke è progettato per funzionare in ambienti distribuiti attraverso l'integrazione con **libcluster**, una libreria che gestisce la formazione e il mantenimento di cluster di nodi.

In un'architettura multi-nodo:

- Ogni nodo mantiene la propria copia locale in ETS per le letture veloci
- Le modifiche alla struttura (nuovo Arke, cambio di Parameters) vengono propagate al cluster
- I nodi si scoprono e si sincronizzano automaticamente, secondo la strategia di clustering configurata

Questo significa che Arke può vivere in ambienti dinamici — container che si avviano e si spengono, istanze che scalano in risposta al carico — senza richiedere configurazione manuale dei nodi o interruzioni di servizio.

La scalabilità non è un'aggiunta: è una conseguenza naturale della piattaforma su cui Arke è costruito.

---

## Il Modello Complessivo

I tre livelli di persistenza si compongono in modo coerente:

| Livello | Tecnologia | Cosa contiene | Caratteristica |
|---|---|---|---|
| Memoria di lavoro | ETS | Struttura degli Arke, configurazioni | Velocissima, locale al nodo |
| Persistenza primaria | PostgreSQL (via connettore) | Tutti i dati applicativi | Fonte di verità, duratura |
| Coordinamento cluster | libcluster | Sincronizzazione tra nodi | Trasparente, automatica |

Ogni livello fa una cosa sola e la fa bene. La complessità della loro interazione è gestita da Arke, non dall'applicazione che lo usa.

---

## Non devi pensarci

Arke si occupa di salvare i dati, tenerli veloci e sincronizzarli tra i nodi. Tu lavori con i tuoi dati, lui si preoccupa del resto, this is the deal.

---

*Torna all'indice: [00 — Indice](./00_indice.md)*
