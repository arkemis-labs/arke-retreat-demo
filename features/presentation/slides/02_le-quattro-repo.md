<!-- discorso: ../discorsi/02_le-quattro-repo.md · ritmo: veloce -->

# Le quattro repo

- **`arke`** — il core: meta-modello, manager, query, hook
- **`arke-postgres`** — persistenza su PostgreSQL
- **`arke-auth`** — identità: utenti, autenticazione, permessi
- **`arke-server`** — layer HTTP (Phoenix)

Direzione delle dipendenze — il core non dipende da nessuno:

```
arke-server ─┬─────────────▶ arke
             ├─▶ arke-postgres ─▶ arke
             └─▶ arke-auth     ─▶ arke
```
