# Slide 2 — Le quattro repo

*Ritmo: Veloce. Testo da dire; le slide verranno adattate dopo.*

---

Prima cosa pratica: Arke non è un monolite. Sono quattro pacchetti, quattro repo, ognuna con una responsabilità precisa.

`arke` è il core: il meta-modello, i manager, le query, gli hook. È puro dominio — non sa come i dati vengano salvati, né come vengano esposti.

`arke-postgres` è la persistenza: traduce le operazioni del meta-modello in SQL su PostgreSQL.

`arke-auth` è l'identità: utenti, autenticazione, permessi.

`arke-server` è il layer HTTP: espone tutto via API, con Phoenix.

Tenete a mente questi quattro nomi, perché torneranno.
