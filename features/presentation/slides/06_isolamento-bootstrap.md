<!-- discorso: ../discorsi/06_isolamento-bootstrap.md · ritmo: veloce -->

# Multi-tenancy = uno schema per Project

- Creare un Project → `CREATE SCHEMA` + copia di `arke_unit` / `arke_link`
- Ogni tenant ha le sue tabelle, nel suo schema → **isolamento totale**

**`arke_system`** — lo schema di bootstrap
- contiene il meta-modello di base (meta-arke, tipi, gruppi di sistema)
- i manager fanno **fallback** qui quando non trovano in locale

> Il multi-tenancy non è un'aggiunta: è come Arke usa gli schemi di Postgres.
