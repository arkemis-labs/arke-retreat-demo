# Slide 3 — La rivelazione: arke_unit

*Ritmo: Soffermarsi. Testo da dire; le slide verranno adattate dopo.*

---

E qui arriva la prima rivelazione, quella che vi chiedo di non dimenticare per il resto della presentazione.

Nella prima parte vi hanno parlato di Arke, Unit, Parameter, Group, Project come concetti distinti. Sul database... sono tutti la stessa cosa. Sono tutti righe della stessa tabella: `arke_unit`.

Lasciatemelo dire in modo netto: il tipo e l'istanza vivono nella stessa tabella. Non c'è una tabella per i "modelli" e una per i "dati". C'è una tabella sola.

Come fa il sistema a distinguerli? Una colonna: `arke_id`. Dice di che cosa quella riga è un'istanza.

Facciamo l'esempio più semplice. Definiamo il tipo "frutto". "frutto" è una riga di `arke_unit`, e il suo `arke_id` vale... `arke`. Perché "frutto" è a sua volta un'istanza del meta-arke "arke": è la definizione di un tipo.

Poi creo una mela. Anche "mela" è una riga di `arke_unit`, ma il suo `arke_id` vale "frutto". Perché la mela è un'istanza di frutto.

Stessa tabella. Cambia solo l'`arke_id`.

E le colonne sono poche, e sono sempre quelle: un `id`, l'`arke_id`, due campi JSONB — `data` e `metadata` — e i timestamp. Gli attributi veri e propri, la label, i valori, stanno dentro `data`.

Questo è il cuore del meta-modello, ed è il motivo per cui in Arke "il modello è modellabile": perché un modello è un dato come tutti gli altri. Lo crei, lo modifichi, lo cancelli esattamente come qualsiasi altra riga.
