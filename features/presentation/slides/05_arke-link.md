<!-- discorso: ../discorsi/05_arke-link.md · ritmo: soffermarsi -->

# `arke_link`: una tabella, tutte le relazioni

PK = (`parent_id`, `child_id`) · niente `id`, niente timestamp · cascade
Sempre direzionato: **parent → child**

La colonna **`type`** ne cambia il significato:

| type | relazione |
|---|---|
| `parameter` | Arke → i suoi campi |
| `group` | Arke → gruppo |
| `permission` | ruolo → Arke |
| `category`, `author`, … | dominio |
| `link` | autonoma generica (N:N) |

> Struttura, topologia, sicurezza — **tutto passa da qui.**
