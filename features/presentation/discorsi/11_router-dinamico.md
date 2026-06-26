# Slide 11 — Il router dinamico

*Ritmo: Soffermarsi. Testo da dire; le slide verranno adattate dopo.*

---

E ora il pezzo che, secondo me, è il più bello di tutta la nostra parte.

Il collega vi ha promesso "API automatiche": definisci un tipo, e l'API c'è. Vi faccio vedere come, concretamente, è possibile.

In un sistema normale, per ogni nuova entità scrivete un controller e scrivete le rotte. Prodotto? Un controller. Ordine? Un altro controller. Cliente? Un altro ancora.

In Arke, no. Il router è dinamico sull'`arke_id`. C'è un solo set di rotte che serve ogni Arke esistente. Guardate: `POST` su `/:arke_id/unit` crea una Unit. `GET` su `/:arke_id/unit/:unit_id` ne legge una. E così via per update, delete, lista. Quella `:arke_id` è una variabile.

Quindi quando definite l'Arke "frutto", non scrivete una riga di routing: `POST /frutto/unit` funziona già. Definite "ordine"? `POST /ordine/unit` funziona già. Lo stesso identico set di rotte serve qualunque tipo abbiate inventato — e qualunque tipo inventerete domani. Vale anche per la topologia, per i gruppi, per le struct.

E come fanno cose come il tenant e i permessi a non essere riscritte in ogni endpoint? Con una pipeline di plug, che ogni richiesta attraversa. Un plug, `GetProject`, legge un header e capisce su quale tenant — su quale schema — state lavorando. Un secondo, l'autenticazione con Guardian, carica il Member dal token. Un terzo, `Permission`, calcola i permessi: o vi blocca con un 403, o decide quali dati potete vedere. Tenant e sicurezza sono trasversali, gestiti una volta sola, non endpoint per endpoint. -> TODO - rielaborare per semplificare

E se a un certo punto vi serve cambiare cosa fa `POST /ordine/unit`? Non toccate né il router né il controller: implementate un hook sull'Arke "ordine". Stesso endpoint, comportamento specifico per quel tipo. Ed è esattamente il ponte verso le prossime due slide.
