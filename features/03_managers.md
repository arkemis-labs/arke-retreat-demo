# Arke — Managers

I "manager" sono i moduli che mediano l'accesso agli elementi del sistema. Si dividono in due
famiglie:

| manager | ruolo | stato |
|---|---|---|
| `ArkeManager` | registro delle definizioni di Arke | ETS (in-memory) |
| `ParameterManager` | registro dei Parameter | ETS (in-memory) |
| `GroupManager` | registro dei Group | ETS (in-memory) |
| `UnitManager` | macro base dei tre registri sopra | — |
| `QueryManager` | CRUD + query relazionali / topologiche | → DB |
| `LinkManager` | mutazioni della topologia (`arke_link`) | → DB |
| `StructManager` | encode / decode Unit ↔ struct (JSON) | — |
| `FileManager` | cache degli URL firmati dei file | ETS (cache con scadenza) |

---

## I registri in-memory (ETS)

Le definizioni del meta-modello — Arke, Parameter, Group — sono i dati **più letti** del sistema:
ogni operazione su una Unit deve verificarne la conformità col tipo. Leggerle dal database ad ogni
richiesta sarebbe un collo di bottiglia, perciò vivono in **ETS**, dietro un GenServer
(vedi *Persistenza → ETS*).

### `UnitManager` — la base

`Arke.Boundary.UnitManager` è una macro: `use Arke.Boundary.UnitManager` + `manager_id(:nome)`
genera un GenServer con una tabella ETS dedicata. I tre registri ne ereditano l'API comune:

- `get/2`, `get_all/1` — lettura;
- `create/1..3`, `update/2..3`, `remove/1..2` — mantengono l'ETS allineato;
- `call_func/3..4` — invoca un hook o una funzione sull'elemento (vedi [Codice](./04_codice.md));
- `get_link/..`, `add_link/..`, `remove_link/..`.

**Fallback sul project**: `get(id, project)` cerca prima nello schema del project, poi in
`:arke_system`. È così che il meta-modello di sistema resta visibile da ogni project senza essere
duplicato.

### `ArkeManager`

Registro delle **definizioni di Arke**. Oltre all'API base:

- `get_parameters/1` — i Parameter associati a un Arke (risolti dai link `type: "parameter"`, con
  gli [override in metadata](./01_database.md#override-in-metadata) già applicati);
- `get_parameter/3` — un singolo parameter risolto;
- `update_parameter/4`.

```elixir
alias Arke.Boundary.ArkeManager

# Uso tipico: recuperare la definizione prima di operare o leggere i parameter in un hook
arke   = ArkeManager.get(:frutto, :my_project)         # %Unit{} (la definizione)
params = ArkeManager.get_parameters(arke)              # [%Unit{}, ...]
peso   = ArkeManager.get_parameter(arke, :my_project, :peso)

# Invocare un hook/funzione dell'Arke (è ciò che QueryManager fa dietro le quinte)
{:ok, unit} = ArkeManager.call_func(arke, :before_create, [arke, unit])
```

### `ParameterManager`

Registro delle **definizioni di Parameter** (`manager_id(:parameter)`). Custodisce i
[tipi](./01_database.md#tipi-di-parameter) e i vincoli intrinseci usati in validazione.

```elixir
alias Arke.Boundary.ParameterManager

# Recuperare la definizione di un parameter (es. per costruire un filtro o validare un valore)
peso = ParameterManager.get(:peso, :my_project)        # %Unit{} (il parameter)
```

### `GroupManager`

Registro dei **Group**. In più:

- `get_arke_list/1` — gli Arke contenuti nel gruppo;
- `get_groups_by_arke/2` — i gruppi a cui un Arke appartiene (usato per invocare gli
  [hook di gruppo](./04_codice.md#hook));
- `get_parameters/1`.

```elixir
alias Arke.Boundary.GroupManager

# Gli Arke contenuti in un gruppo
arke_list = GroupManager.get(:oauth_provider, :arke_system).data.arke_list

# I gruppi a cui un Arke appartiene → su questi si lanciano gli hook *_unit_* di gruppo
groups = GroupManager.get_groups_by_arke(arke)
```

---

## I manager operativi

### `QueryManager`

CRUD + query relazionali e topologiche. È il più usato e ha un documento dedicato:
**[Query Manager](./02_query_manager.md)**.

### `LinkManager`

Muta la **topologia**, scrivendo / aggiornando / eliminando righe in
[`arke_link`](./01_database.md#link):

- `add_node(project, parent, child, type \\ "link", metadata \\ %{})`
- `update_node/5`
- `delete_node/5`

Accetta sia `%Unit{}` sia id stringa. È il meccanismo dietro i
[Link autonomi](./01_database.md#link) e, indirettamente, dietro i link strutturali
(`QueryManager` lo chiama quando si scrive un parameter di tipo `link`).

```elixir
alias Arke.LinkManager

# Link strutturale: lega il parameter "peso" all'arke "frutto"
LinkManager.add_node(:my_project, "frutto", "peso", "parameter", %{required: true})

# Link autonomo di dominio (relazione N:N tra due Unit)
LinkManager.add_node(:my_project, "cliente_01", "ordine_42", "ordine", %{})

# Rimuovere un link
LinkManager.delete_node(:my_project, "cliente_01", "ordine_42", "ordine")
```

### `StructManager`

Traduce le Unit da/verso una rappresentazione serializzabile (la "struct", tipicamente JSON per
l'API):

- `encode/2` — Unit → struct, con opzioni `load_links`, `load_values`, `load_files`;
- `decode/4` — dati grezzi → Unit;
- `get_struct/..`, `validate_data/4`.

È il layer che [`arke_server`](./04_codice.md#il-router) usa per restituire le Unit.

```elixir
alias Arke.StructManager

# Serializzare una Unit (o una lista) per la risposta API
json  = StructManager.encode(unit,  type: :json)
jsons = StructManager.encode(units, load_links: true, load_values: true, type: :json)

# Dal payload grezzo (es. body di una request) alla Unit
unit = StructManager.decode(:my_project, "frutto", payload, :json)   # %Unit{}
```

### `FileManager`

Cache (ETS) degli **URL firmati** dei file, con chiave `{unit_id, project}` e scadenza; le voci
scadute vengono ripulite periodicamente. Evita di rifirmare lo stesso file ad ogni richiesta.

```elixir
alias Arke.Boundary.FileManager

# Pattern tipico: prima cerca in cache, altrimenti firma e memorizza
case FileManager.get(unit_id, project) do
  nil    -> FileManager.add(unit, sign_url(unit))   # miss → ricalcola e salva
  cached -> cached                                  # hit  → riusa
end

FileManager.remove(unit_id, project)                # invalida la voce
```

---

## Creare Arke / Parameter / Link da CLI

Aprendo una console (`iex -S mix`) si crea il modello a mano. Arke e Parameter sono **Unit di un
meta-arke**, quindi nascono con `QueryManager.create` (vedi [Query Manager](./02_query_manager.md));
i Link con `LinkManager`.

```elixir
alias Arke.QueryManager
alias Arke.Boundary.ArkeManager
alias Arke.LinkManager

# 1. PARAMETER — una Unit dell'arke che ne è il tipo (qui :integer)
integer = ArkeManager.get(:integer, :my_project)
{:ok, _peso} = QueryManager.create(:my_project, integer, id: "peso", label: "Peso", min: 0)
# l'hook on_unit_create registra il parameter nel ParameterManager (ETS)

# 2. ARKE — una Unit del meta-arke :arke
arke_model = ArkeManager.get(:arke, :my_project)
{:ok, _frutto} = QueryManager.create(:my_project, arke_model, id: "frutto", label: "Frutto")
# l'hook on_create registra l'Arke nell'ArkeManager (ETS)

# 3. LINK — lega il parameter "peso" all'arke "frutto" (link strutturale)
LinkManager.add_node(:my_project, "frutto", "peso", "parameter", %{required: true})
```

Per popolare un intero project in modo **dichiarativo** (dai file di registry) c'è invece il mix
task dedicato:

```bash
mix arke.seed_project --project my_project   # un project
mix arke.seed_project --all                  # tutti i project sul database
```

---

*Vedi anche: [Query Manager](./02_query_manager.md) · [Codice](./04_codice.md) ·
[Autenticazione](./05_autenticazione.md) · [Database](./01_database.md)*
