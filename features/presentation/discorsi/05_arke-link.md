# Slide 5 — arke_link: una tabella per ogni relazione

*Ritmo: Soffermarsi. Testo da dire; le slide verranno adattate dopo.*

---

Abbiamo detto due tabelle. La prima è `arke_unit`, e tiene tutte le entità. La seconda è `arke_link`, e tiene tutte le relazioni. Tutte.

Un Link è una relazione tra due Unit. La struttura è semplicissima: non ha nemmeno un `id`, non ha timestamp. La sua chiave primaria è la coppia `parent_id`, `child_id`. Più due colonne: un `type` e un `metadata`.

Ogni Link è direzionato: va da un parent a un child. Le foreign key vanno in cascade — cancelli una Unit, spariscono i suoi link.

Ma la colonna su cui voglio fermarmi è `type`. Perché è quella che fa una cosa che, secondo me, è il vero colpo di genio della struttura.

Nella prima parte vi hanno parlato di relazioni diverse: i link strutturali, dentro la definizione di un Arke; i link autonomi, le relazioni N a N; l'appartenenza a un gruppo. E tra poco vedremo anche i permessi.

Bene. Sul database sono tutti la stessa cosa: righe di `arke_link`. Cambia solo il `type`.

`type` uguale a `parameter`: è il link che lega un Arke ai suoi campi. `type` uguale a `group`: è l'appartenenza di un Arke a un gruppo. `type` uguale a `permission`: è un permesso — lo vedremo. `type` uguale a un valore vostro, "categoria", "autore": è una relazione di dominio. E il default, `link`: la relazione autonoma generica.

Una tabella. Una colonna che ne cambia il significato. La struttura, la topologia, la sicurezza — tutto passa da qui.

E quando interrogate il grafo — "tutto ciò che è collegato a questo cliente" — potete dire al sistema di seguire solo certi `type` e ignorare il resto. Ma alle query ci arriviamo tra due slide.
