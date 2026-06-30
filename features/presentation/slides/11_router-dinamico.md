<!-- discorso: ../discorsi/11_router-dinamico.md · ritmo: soffermarsi -->

# Il router è dinamico su `:arke_id`

Un solo set di rotte serve **ogni** Arke. Nessun controller per tipo.

```
POST   /:arke_id/unit            → crea
GET    /:arke_id/unit/:unit_id   → leggi
PUT    /:arke_id/unit/:unit_id   → aggiorna
DELETE /:arke_id/unit/:unit_id   → elimina
```

Definisci `frutto` → `POST /frutto/unit` **funziona già.**
Idem `ordine`, e qualunque arke inventerai domani.

---

# Trasversale a ogni richiesta

**Pipeline di plug:**

```
GetProject   →   Auth          →   Permission
(quale tenant)   (chi sei)         (cosa puoi fare)
```

> Cambiare un comportamento? Un **hook** sull'Arke — non il router, non il controller.
