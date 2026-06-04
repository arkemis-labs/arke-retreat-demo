# Getting Started: Arke Framework Demo

**Reference:** [Arke Documentation - Create Application](https://docs.arkehub.com/docs/create-application)

## 1. Roadmap della Demo

| Fase | Cosa si mostra | Concetto chiave |
|------|----------------|-----------------|
| 1    | Slide/intro    | Filosofia Arke  |
| 2    | seed | Project setup |
| 3    | QueryManager (Arke + Unit + Link) | Definizione Modello + Topologia |
| 4    | Swagger live   | API RESTful autogenerata |
| 6    | Custom function Elixir | Estensibilità backend |

---

## 2. Out-of-the-Box: API Autogenerata

Aprire Swagger/REST e mostrare che, **senza scrivere un singolo controller**, sono già disponibili gli endpoint completi. Filtri, sorting, paginazione e auth sono tutto incluso di default come query params.

```http
GET  /api/spaceship/lib/room/unit            # Lista stanze 
POST /api/spaceship/lib/mission/unit         # Crea missione 
# etc...

Filtri, sorting, paginazione — tutto incluso come query params.

## Creazione entità di base

**Arke** = la struttura/schema

- Creiamo un arke X
- Creiamo i parametri dell'Arke A,B,C
- Creiamo delle unit
- Colleghiamo le unit appena create ad altre unit
- Creiamo dei gruppi con regole comuni

**RESTful API autogenerate** per ogni Unit → sorting, filtering, auth già inclusi

- Utilizzare hook su operazioni crud
- Creare custom functions

### Tools

Comando di export del DB

`source .env && mix arke.export_data --project spaceship --splitfile`

Creazione arke

```elixir
arke_model = Arke.Boundary.ArkeManager.get(:arke, :arke_system)
{:ok, arke} = Arke.QueryManager.create(:spaceship, arke_model, %{id: "room", label: "Room"})
```

## Database

### ARKE

- Group `voyajer`
  - Group `crew_member`
    - Arke `captain`
      - name
      - Link `link_room` (`captain` → `room`)
      - `arke_system_user`
    - Arke `technician`
      - name
      - Link `link_room` (`technician` → `room`)
      - `arke_system_user`
  - Arke `passenger`
    - name
    - Link `link_room` (`passenger` → `room`)
    - `arke_system_user`
- Arke `room`
  - name
  - description
  - restricted
- Arke `system`
  - name
  - status
  - Link `link_room` (`system` → `room`)
- Arke `mission`
  - name
  - description
  - expiration_datetime

### LINK

vedi seed

### UNIT

vedi seed

### PERMISSIONS

vedi seed

### Metodi

- Creazione di una matricola associata ad un utente, progressiva a prescindere dal ruolo.

`mix archive.install hex arke_new`
`mix arke.new .`
`mix deps.get`

Add a `.env` file in the root of the project to set the Database env variables:

```js
export DB_NAME="arke_db"
export DB_HOSTNAME="127.0.0.1"
export DB_USER="postgres"
export DB_PASSWORD="postgres"
```

Run `source .env` to apply your environment variables

Create Database and define arke_system schema using ArkePostgres defaults:

```shell
mix arke.init
```

```shell
mix arke_postgres.create_project --id starship
mix arke.seed_project --project starship
```

```shell
mix arke_postgres.create_member --project starship --username {YOUR_USERNAME} --password {YOUR_PASSWORD} --email {YOUR_EMAIL}
```

`iex -S mix phx.server` o più completo per debug `source .env && iex --dbg pry -S mix phx.server`

Arke Backend si avvierà in <http://localhost:4000>
