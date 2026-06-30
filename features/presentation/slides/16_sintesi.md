<!-- discorso: ../discorsi/16_sintesi.md · ritmo: veloce -->

# Concetto → Implementazione

| concetto                    | implementazione                       |
| --------------------------- | ------------------------------------- |
| tipi e istanze              | righe di `arke_unit`                  |
| relazioni, gruppi, permessi | righe di `arke_link` (colonna `type`) |
| struttura del sistema       | il Registry (in git)                  |
| API                         | un router dinamico                    |
| logica di dominio           | hook + funzioni custom                |
| identità e sicurezza        | ancora Unit e Link                    |

> **Due tabelle, un registry, un router.**
> Il sistema si adatta al dominio, non il contrario.