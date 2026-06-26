# Slide 8 — QueryManager: una sola API

*Ritmo: Medio. Testo da dire; le slide verranno adattate dopo.*

---

Ok: abbiamo le entità in `arke_unit`, le relazioni in `arke_link`. Come ci leggiamo e ci scriviamo? Con un solo strumento: il QueryManager.

È il punto d'accesso unico per leggere e scrivere Unit. E fa una cosa importante: costruisce la query sul meta-modello, ma delega l'esecuzione al connettore — ad `arke-postgres`. Lui resta indipendente dal database. Se domani cambiate database, l'API resta identica.

Con lo stesso strumento coprite i due modi di leggere i dati di cui vi ha parlato il collega. Il modo relazionale: "dammi tutti i frutti con peso maggiore di cento". E il modo topologico: parti da una Unit e segui i suoi link — è la funzione `link`, con la profondità, la direzione, e il `type` da seguire.

Per le letture comuni ci sono delle scorciatoie — `get_by` per un singolo elemento, `filter_by` per una lista. E per i casi più complessi le query sono componibili: ogni funzione restituisce una query, e voi le concatenate in pipeline — apri la query, aggiungi i filtri, ordina, pagina, esegui. Niente tocca il database finché non chiamate l'esecutore, alla fine.

E un'ultima cosa, che è comodissima: una `create` o un `update` non scrivono solo la riga. Fanno scattare la validazione, gli hook del ciclo di vita, e la gestione automatica dei link strutturali. Una sola chiamata, e il sistema tiene tutto coerente.
