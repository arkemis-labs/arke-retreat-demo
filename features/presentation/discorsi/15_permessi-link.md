# Slide 15 — I permessi sono Link

*Ritmo: Medio. Testo da dire; le slide verranno adattate dopo.*

---

E rieccoci ad `arke_link`, come promesso. Perché anche un permesso è un Link. Una riga di `arke_link` con `type` uguale a `permission`.

Leggiamola insieme: il parent è il ruolo — `member_public`, cioè tutti, oppure un ruolo specifico, tipo `installer` o `super_admin`. Il child è l'Arke su cui il permesso si applica. E il metadata contiene i permessi veri e propri.

Quali permessi? Quattro flag booleani, uno per verbo: `get`, `post`, `put`, `delete`. Quando arriva una richiesta, il plug `Permission` traduce il metodo HTTP nel flag corrispondente: se è `false`, 403, fine. I permessi effettivi nascono dalla fusione di una base pubblica con quelli del ruolo del member. Con un'eccezione: il `super_admin` ha tutto true.

Ma c'è un livello più fine, ed è la parte elegante. Un permesso può portare anche un `filter`: non decide solo *se* puoi fare un'operazione, ma *su quali* righe. L'installatore vede solo gli ordini di cui è autore. L'utente vede solo le proprie pratiche. È sicurezza a livello di singola riga, e viene applicata automaticamente alla query dell'endpoint.

Quindi tre livelli, che si sommano: il Project — esisti come member attivo qui? L'Arke — i quattro flag, cosa puoi fare su questo tipo. E la Unit — il filtro, quali righe specifiche. Tutto espresso come dati e link, non come codice di autorizzazione sparso ovunque.
