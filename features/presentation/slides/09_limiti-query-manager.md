<!-- discorso: ../discorsi/09_limiti-query-manager.md · ritmo: veloce -->

# QueryManager — i limiti

- Operatori **predefiniti** (`=`, `LIKE`, range, `IN`, `IS NULL`)
- Ogni scrittura passa per gli **hook** → a volte lento
- **Niente bulk** (insert / update / delete massivi)
- Update o delete → **prima una select**
- Ritorna **sempre l'intera Unit** (no select parziale)

> Casi estremi → si scende a **Ecto** (`pseudo_query`): join, aggregati, window function.
