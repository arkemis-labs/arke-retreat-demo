# Arke — Mappa dei Moduli

> Indice tecnico di `features/`. Per la visione d'insieme e i concetti, vedi
> [filosofia/02 — Architettura](../filosofia/02_architettura.md).

Arke non è un monolite: è un insieme di **repo** (pacchetti Elixir) componibili, ognuna con una
responsabilità precisa, adottabili in modo selettivo. Il backend si regge su quattro repo.

---

## Le quattro repo

| repo | responsabilità | app / namespace | approfondimento |
|---|---|---|---|
| `arke` | il **core**: meta-modello, manager, query, hook | `:arke` · `Arke.*` | [01](./01_database.md) · [02](./02_query_manager.md) · [03](./03_managers.md) · [04](./04_codice.md) |
| `arke-postgres` | **connettore di persistenza** su PostgreSQL | `:arke_postgres` · `ArkePostgres.*` | [01](./01_database.md) |
| `arke-auth` | **identità, autenticazione, permessi** | `:arke_auth` · `ArkeAuth.*` | [05](./05_autenticazione.md) |
| `arke-server` | **API HTTP** (Phoenix) sul meta-modello | `:arke_server` · `ArkeServer.*` | [04](./04_codice.md) |

> **Sui nomi**: il nome ufficiale della repo/pacchetto usa il trattino — `arke-server`,
> `arke-postgres`, `arke-auth`. L'app Elixir e la cartella usano l'underscore (`arke_server`), il
> modulo il PascalCase (`ArkeServer`). Sono la stessa cosa in contesti diversi.

---

## Responsabilità

**`arke` — il core.** Definisce il meta-modello (Arke, Unit, Parameter, Link, Group, Project), i
[manager in-memory](./03_managers.md) (ETS), il [Query Manager](./02_query_manager.md) e gli
[hook del ciclo di vita](./04_codice.md#hook). È puro dominio: **non sa** come i dati vengano
salvati né come vengano esposti. Definisce soltanto il *contratto* di persistenza. Non dipende da
nessuna delle altre repo.

**`arke-postgres` — la persistenza.** Implementa quel contratto su PostgreSQL: traduce le
operazioni del meta-modello in SQL, gestisce le tabelle [`arke_unit` / `arke_link`](./01_database.md)
e l'isolamento [uno schema per Project](./01_database.md#project). È un *adattatore*, non
un'estensione: dipende solo da `arke`.

**`arke-auth` — l'identità.** Aggiunge i soggetti delle operazioni: [User e Member](./05_autenticazione.md),
i token JWT (Guardian) e i [permessi espressi come Link](./05_autenticazione.md#permessi). Dipende
solo da `arke`.

**`arke-server` — il layer API.** Espone tutto via HTTP con Phoenix: [router dinamico](./04_codice.md#il-router)
su `:arke_id`, controller, serializzazione via `StructManager`. Sta in cima allo stack: dipende da
`arke`, `arke-postgres` e `arke-auth`.

---

## Direzione delle dipendenze

Le frecce indicano "**dipende da**":

```
arke-server ──┬─────────────▶ arke
              ├─▶ arke-postgres ─▶ arke
              └─▶ arke-auth      ─▶ arke
```

- `arke` è il **fondamento**: non dipende da nessuna delle altre repo;
- `arke-postgres` e `arke-auth` estendono il core ortogonalmente, dipendendo **solo** da `arke`;
- `arke-server` è in **cima** e le compone tutte.

Il verso non è casuale: persistenza, identità e trasporto dipendono dal core, **mai il contrario**.
È la stessa inversione di dipendenza descritta in *Persistenza* — il core resta ignaro di dove e
come vive il dato.

---

## Quale documento leggere

| documento | di cosa parla | repo |
|---|---|---|
| [01 — Database](./01_database.md) | rappresentazione su DB: `arke_unit`, `arke_link`, tipi | `arke` · `arke-postgres` |
| [02 — Query Manager](./02_query_manager.md) | leggere e scrivere Unit, query relazionali e topologiche | `arke` |
| [03 — Managers](./03_managers.md) | i registri ETS e i manager operativi | `arke` |
| [04 — Codice](./04_codice.md) | router, hook, funzioni custom | `arke-server` · `arke` |
| [05 — Autenticazione](./05_autenticazione.md) | User/Member, JWT, permessi, pipeline | `arke-auth` |

---

*Visione concettuale: [filosofia/02 — Architettura](../filosofia/02_architettura.md) ·
[filosofia/04 — Persistenza](../filosofia/04_persistenza.md)*
