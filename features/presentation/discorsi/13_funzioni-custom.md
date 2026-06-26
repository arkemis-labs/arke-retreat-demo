# Slide 13 — Funzioni custom

*Ritmo: Medio. Testo da dire; le slide verranno adattate dopo.*

---

Il CRUD generico è comodo, ma un dominio reale non è fatto solo di "crea, leggi, aggiorna, cancella". Un ordine si "conferma". Un documento si "pubblica", si "archivia". Queste non sono operazioni standard: sono il vocabolario specifico del vostro dominio.

Per questo ci sono le funzioni custom. Dove gli hook reagiscono a eventi standard, le funzioni custom aggiungono operazioni nuove.

Si scrivono come semplici funzioni pubbliche nel modulo dell'Arke. E vengono esposte automaticamente come endpoint dedicati: `/:arke_id/function/:nome_funzione` a livello di tipo, oppure sulla singola Unit. Quindi la funzione "conferma" sull'ordine diventa, da sola, un endpoint `/ordine/unit/.../function/conferma`.

Dentro la funzione avete tutto quello che vi serve della richiesta — i parametri, il body, il member autenticato — che il sistema vi inietta. E non dovete restituire una risposta HTTP a mano: restituite un dato, e il controller lo trasforma nella risposta. Potete dire "ok, questo contenuto, status 200", oppure restituire un file, oppure un errore.

Quindi, oltre al CRUD automatico, avete un modo pulito per esprimere le azioni che contano davvero nel vostro dominio.
