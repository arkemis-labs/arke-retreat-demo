# Arke — Query Manager

## Cos'è

`Arke.QueryManager` è il **punto d'accesso unico** per leggere e scrivere Unit. Costruisce query
sul meta-modello (un `project` e, opzionalmente, un `arke`) e **delega l'esecuzione al connettore**
configurato (`arke_postgres`), restando indipendente dal database.

Copre i due modelli di lettura descritti in *Topologia*:

- **relazionale** — filtra le Unit per attributo;
- **topologico** — parte da una Unit e ne percorre i Link (`link/3`).

Ogni operazione porta con sé il `project`: è **project-aware**, quindi lavora sullo schema del
tenant corretto (vedi [Project](./01_database.md#project)).

### Vantaggi

- **Una sola API** per CRUD, query relazionali e traversata del grafo.
- **Integrazione automatica**: una `create` / `update` non scrive solo la riga in `arke_unit`,
  ma innesca validazione, [hook del ciclo di vita](./04_codice.md#hook) e gestione dei
  [link strutturali](./01_database.md#associazione-ad-arke) — i parameter di tipo `link`
  diventano righe in `arke_link`.
- **Componibile**: le funzioni ritornano una `%Arke.Core.Query{}`, quindi si concatenano in
  pipeline prima di essere eseguite.
- **Indipendente dalla persistenza**: la stessa API resta valida anche cambiando connettore.

### Limiti

- Ritorna sempre l'intera unit, non è possibile limitare la quantità di colonne selezionate.
- Gli operatori sono un set predefinito (uguaglianza, `LIKE`, range, `IN`, `IS NULL`).
- Costringe a passare dai lifecycle hook => lentezza che a volte è problematica.
- Assenza di bulk actions per insert, delete e update che su SQL sarebbero molto più rapide.
- Per eseguire update o delete bisogna sempre eseguire prima una select per avere l'intera unit.

Per join custom, aggregati o window function si scende a Ecto tramite `pseudo_query/1`.

---

## Le funzioni

### Scorciatoie di lettura — `get_by` / `filter_by`

Per le letture comuni, senza costruire la query a mano:

```elixir
# Una sola Unit (o nil)
mela = QueryManager.get_by(project: :my_project, arke_id: "frutto", id: "mela")

# Lista di Unit che soddisfano i criteri
pesanti = QueryManager.filter_by(project: :my_project, arke_id: "frutto", peso__gte: 100)
```

I criteri usano la notazione `parametro__operatore: valore` (vedi [Operatori](#operatori-di-confronto)).

### Scrittura — `create` / `update` / `delete`

```elixir
frutto = Arke.Boundary.ArkeManager.get(:frutto, :my_project)   # definizione (vedi Managers)

{:ok, mela} = QueryManager.create(:my_project, frutto, nome: "Mela", peso: 150)
{:ok, mela} = QueryManager.update(mela, peso: 160)
{:ok, nil}  = QueryManager.delete(:my_project, mela)
```

Ognuna attraversa la catena di validazione e hook descritta in
[Codice → Hook](./04_codice.md#hook).

### Funzioni componibili (query building)

Costruiscono e affinano una `%Query{}`; **nulla tocca il DB** finché non si chiama un esecutore.

**`query/1`** — apre una query su un project (e opzionalmente un arke):

```elixir
q = QueryManager.query(project: :my_project, arke: :frutto)
```

**`where/2`** — aggiunge filtri in `AND` con la notazione `parametro__operatore`:

```elixir
q |> QueryManager.where(peso__gte: 100, nome__icontains: "mel")
```

**Esecutori — `all` / `one` / `count` / `raw`** — chiudono la pipeline:

| funzione | ritorna |
|---|---|
| `all/1` | `[%Unit{}]` |
| `one/1` | `%Unit{}` o `nil` |
| `count/1` | `integer` |
| `raw/1` | la query come SQL (`{sql, params}`) — per ispezione/debug |

```elixir
q |> QueryManager.where(peso__gte: 100) |> QueryManager.all()
```

**`link/3`** — query topologica: parte da una Unit e ne segue i Link.

```elixir
q = QueryManager.query(project: :my_project)
u = QueryManager.get_by(project: :my_project, id: "cliente_01")

QueryManager.link(q, u, depth: 2, direction: :child, type: "ordine")
|> QueryManager.all()
```

Opzioni: `direction` (`:child` default / `:parent`), `depth` (default 500), `type` (limita ai
Link di quel tipo).

**`order/3` · `offset/2` · `limit/2`** (+ `pagination/3`):

```elixir
q
|> QueryManager.order(:peso, :desc)
|> QueryManager.offset(0)
|> QueryManager.limit(20)
|> QueryManager.all()

# In alternativa, conteggio + pagina in un colpo solo:
{count, elements} = QueryManager.pagination(q, 0, 20)
```

**`filter` — come si creano i filtri**

Nella forma diretta, `filter/4` aggiunge una singola condizione `parametro operatore valore`:

```elixir
# frutti con peso >= 100
q |> QueryManager.filter(:peso, :gte, 100)
```

Un quinto argomento opzionale `negate` (default `false`) nega la condizione.

`where/2` copre il caso `AND` semplice. Per logiche booleane più ricche si combinano `and_/3`,
`or_/3` e `conditions/1`:

```elixir
q
|> QueryManager.and_(false, QueryManager.conditions(peso__gte: 100, peso__lte: 200))
|> QueryManager.or_(false,  QueryManager.conditions(nome__icontains: "bio"))
```

- `conditions/1` produce una lista di `%BaseFilter{}` dalla notazione `parametro__operatore`;
- il parametro `negate` (`true` / `false`) nega l'intero blocco.

**Filtri nestati** — si può filtrare attraversando un parameter di tipo `link`, indicando il
**path** col punto:

```elixir
# frutti la cui categoria collegata ha label "Bio"
QueryManager.filter(q, "categoria.label", :eq, "Bio")
```

**`pseudo_query/1`** — interop con Ecto: invece di eseguire, restituisce la `%Ecto.Query{}`
generata, da comporre con `Ecto.Query` e `ArkePostgres.Repo` per i casi che gli operatori
predefiniti non coprono:

```elixir
ecto_q =
  QueryManager.query(project: :my_project, arke: :frutto)
  |> QueryManager.where(peso__gte: 100)
  |> QueryManager.pseudo_query()
```

---

## Operatori di confronto

Si usano come suffisso sulla chiave (`parametro__operatore: valore`) in `where`, `conditions`,
`get_by`, `filter_by`:

| operatore | significato | SQL |
|---|---|---|
| `eq` | uguale | `=` |
| `contains` / `icontains` | contiene (case sensitive / insensitive) | `LIKE %v%` |
| `startswith` / `istartswith` | inizia per | `LIKE v%` |
| `endswith` / `iendswith` | finisce per | `LIKE %v` |
| `lt` / `lte` | minore / minore o uguale | `<` / `<=` |
| `gt` / `gte` | maggiore / maggiore o uguale | `>` / `>=` |
| `in` | appartiene a una lista | `IN` |
| `isnull` | è nullo | `IS NULL` |

Senza suffisso, il default è `eq`.

---

*Vedi anche: [Managers](./03_managers.md) per `ArkeManager` / `LinkManager`,
[Codice](./04_codice.md) per hook e funzioni custom,
[Autenticazione](./05_autenticazione.md) per utenti e permessi.*
