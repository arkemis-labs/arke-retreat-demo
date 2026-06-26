# Slide 4 — Il Parameter e il trucco di persistence

*Ritmo: Soffermarsi. Testo da dire; le slide verranno adattate dopo.*

---

Restiamo sulla stessa tabella, perché c'è un secondo pezzo di magia.

Un Parameter — un campo: "peso", "nome", "descrizione" — anche lui è una riga di `arke_unit`. E qui la cosa elegante: l'`arke_id` di un Parameter è il tipo del suo valore.

Quindi il parametro "peso" ha `arke_id` uguale a `integer`. Il parametro "nome" ha `arke_id` uguale a `string`. Il tipo del campo è scritto nell'`arke_id`. I vincoli — minimo, massimo, lunghezza, obbligatorietà — stanno dentro `data`.

Ora la domanda naturale è: ma se la tabella ha solo quelle quattro o cinque colonne fisse, come fa a contenere il "peso" di un frutto, il "titolo" di un articolo, qualsiasi campo di qualsiasi tipo?

La flessibilità sta tutta nel JSONB di `data`. È così che la stessa tabella regge frutti, articoli, ordini, utenti, qualsiasi cosa — senza una migration.

Un'ultima cosa, veloce, perché il concetto lo avete già sentito: lo stesso parametro può essere riusato su più Arke, e su ogni Arke i suoi vincoli si possono sovrascrivere. "descrizione" può essere facoltativo su un tipo e obbligatorio su un altro, senza ridefinirlo. Ma quello lo sapete già.
