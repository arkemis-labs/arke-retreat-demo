# Arke — Codice: Router, Hook, Funzioni Custom

Dove si scrive il codice di dominio in un sistema Arke. Due meccanismi complementari:

- **Hook** — *reagiscono* agli eventi del ciclo di vita (creazione, aggiornamento, …);
- **Funzioni custom** — *aggiungono* operazioni nuove, esposte come endpoint dedicati.

---

## Il Router

`arke_server` espone il sistema via HTTP con un router Phoenix (`ArkeServer.Router`). La
caratteristica: gli endpoint sono **dinamici su `:arke_id`** — un solo set di route serve *ogni*
Arke definito, senza scrivere controller per tipo.

Gli endpoint — tutti montati sotto lo scope `/lib` — coprono ogni risorsa del meta-modello. I path
con `:arke_id` / `:group_id` sono **dinamici**: la stessa route serve qualunque tipo.

**Unit — CRUD**

| metodo | path | azione |
|---|---|---|
| `POST` | `/:arke_id/unit` | crea una Unit |
| `GET` | `/:arke_id/unit` | lista delle Unit |
| `GET` | `/:arke_id/unit/:unit_id` | dettaglio Unit |
| `PUT` | `/:arke_id/unit/:unit_id` | aggiorna |
| `DELETE` | `/:arke_id/unit/:unit_id` | elimina |
| `GET` | `/unit` | ricerca globale (su tutti gli Arke) |

**Arke — struct e metadati**

| metodo | path | azione |
|---|---|---|
| `GET` | `/:arke_id/struct` | struct (definizione) dell'Arke |
| `GET` | `/:arke_id/count` | conteggio delle Unit |
| `GET` | `/:arke_id/group` | gruppi a cui l'Arke appartiene |
| `GET` | `/:arke_id/unit/:unit_id/struct` | struct di una singola Unit |

**Topologia — Link**

| metodo | path | azione |
|---|---|---|
| `GET` | `/:arke_id/unit/:unit_id/link/:direction` | nodi collegati (`parent` / `child`) |
| `GET` | `/:arke_id/unit/:unit_id/link/:direction/count` | conteggio dei nodi collegati |
| `POST` | `/:arke_id/unit/:unit_id/link/:link_id/:arke_id_two/unit/:unit_id_two` | crea un link tra due Unit |
| `PUT` | `/:arke_id/unit/:unit_id/link/:link_id/:arke_id_two/unit/:unit_id_two` | aggiorna il link |
| `DELETE` | `/:arke_id/unit/:unit_id/link/:link_id/:arke_id_two/unit/:unit_id_two` | elimina il link |

**Parameter**

| metodo | path | azione |
|---|---|---|
| `POST` | `/:arke_id/parameter/:arke_parameter_id` | associa un parameter all'Arke |
| `PUT` | `/:arke_id/parameter/:arke_parameter_id` | aggiorna l'associazione |
| `GET` | `/parameter/:parameter_id` | valore di un parameter di tipo `link` |
| `POST` | `/parameter/:parameter_id` | aggiunge un valore (link) |
| `PUT` | `/parameter/:parameter_id` | aggiorna il valore |
| `DELETE` | `/parameter/:parameter_id/:unit_id` | rimuove il valore |

**Group**

| metodo | path | azione |
|---|---|---|
| `GET` | `/group/:group_id/arke` | gli Arke contenuti nel gruppo |
| `GET` | `/group/:group_id/struct` | struct del gruppo |
| `GET` | `/group/:group_id/unit` | Unit di tutti gli Arke del gruppo |
| `GET` | `/group/:group_id/unit/:unit_id` | dettaglio di una Unit del gruppo |
| `GET` `POST` | `/group/:group_id/function/:function_name` | [funzione custom](#funzioni-custom) di gruppo |

**Funzioni custom**

| metodo | path | azione |
|---|---|---|
| `GET` `POST` | `/:arke_id/function/:function_name` | [funzione](#funzioni-custom) a livello di Arke |
| `GET` `POST` | `/:arke_id/unit/:unit_id/function/:function_name` | [funzione](#funzioni-custom) a livello di Unit |

**Sistema (endpoint fissi, non dinamici)**

| metodo | path | azione |
|---|---|---|
| `GET` `POST` | `/auth/signin` | login |
| `POST` | `/auth/:arke_id/signup` | registrazione |
| `POST` | `/auth/refresh` · `/auth/verify` | refresh / verifica token |
| `POST` | `/auth/change_password` · `/auth/recover_password` · `/auth/reset_password[/:token]` | gestione password |
| `POST` | `/auth/signin/:provider` · `/auth/:member/:provider` | login / creazione member via SSO |
| `GET` `POST` `PUT` `DELETE` | `/arke_project/unit[/:unit_id]` | gestione dei Project |
| `GET` | `/health/ready` · `/live` · `/start` | health check |
| `GET` | `/arke_dev_function/export_arke_db_stucture` | export della struttura DB |

### Le pipeline (plug)

Le richieste passano per delle **pipeline** (plug): `:project` risolve il tenant (`GetProject`,
dall'header `arke-project-key`) e costruisce i filtri; `:auth_api` applica i permessi
(`Permission`); `:get_unit` carica la Unit. Project e permessi sono quindi trasversali, non
riscritti endpoint per endpoint.

A runtime, ogni richiesta attraversa una catena di plug:

| plug | responsabilità |
|---|---|
| `GetProject` | risolve il tenant dall'header **`arke-project-key`** → assegna `:arke_project` |
| `AuthPipeline` | Guardian: `VerifyHeader` (Bearer) → `EnsureAuthenticated` → `LoadResource` (carica il Member dal JWT) |
| `Permission` | calcola i permessi del Member sull'arke richiesto: nega con `403`, oppure assegna il `filter` da applicare alla query |
| `NotAuthPipeline` | per gli endpoint **pubblici** (es. `signin`): richiede *assenza* di autenticazione |

Esiste anche una `ImpersonateAuthPipeline` (doppio token) per l'impersonation, abilitabile da
configurazione.

---

### Override dell'endpoint di un Arke (esempio di flessibilità)

Non serve toccare router o controller per cambiare il comportamento di un'operazione standard:
l'endpoint generico `POST /:arke_id/unit` chiama `QueryManager.create`, che a sua volta esegue gli
**hook** dell'Arke. Implementare `before_create` / `on_create` sull'Arke `ordine` cambia di fatto
cosa fa `POST /ordine/unit` — **stesso endpoint, comportamento specifico per tipo**.

Quando invece serve un'operazione che non è un CRUD, si aggiunge una **funzione custom** (sotto).
E poiché quello di Arke è un normale router Phoenix, l'applicazione ospite può comunque dichiarare
route proprie più specifiche, che hanno precedenza sul match dinamico.

---

## Hook

Un hook è una callback sul **ciclo di vita** di un Arke. Si abilitano con `use Arke.System` e si
sovrascrivono solo quelli che servono (gli altri hanno un default no-op).

Hook a livello di **Arke**:

| momento | hook | uso tipico |
|---|---|---|
| caricamento | `before_load/2`, `on_load/2` | normalizzare i dati letti |
| validazione | `before_validate/2`, `on_validate/2` | regole custom |
| creazione | `before_create/2`, `on_create/2` | arricchimento, effetti collaterali |
| aggiornamento | `before_update/3`, `on_update/3` | coerenza, blocco di modifiche |
| eliminazione | `before_delete/2`, `on_delete/2` | pulizia di relazioni |
| serializzazione | `before_struct_encode/2`, `on_struct_encode/4` | dati derivati nella risposta |

```elixir
defmodule MyProject.Arke.Ordine do
  use Arke.System

  arke id: "ordine" do
    parameter :stato, :string, default: "bozza"
  end

  def on_create(_arke, unit) do
    MyProject.Mailer.invia_conferma(unit)
    {:ok, unit}
  end
end
```

`QueryManager.create` li invoca in sequenza precisa:

```
Unit.load → validate → before_create → before_unit_create (gruppi)
          → persist → on_create → on_unit_create (gruppi)
```

Le varianti `*_unit_*` sono gli **hook a livello di Group** (`use Arke.System.Group`): si applicano
a tutti gli Arke che appartengono al gruppo — il punto giusto per logica condivisa da più tipi.

Gli hook **non** sostituiscono la validazione strutturale (garantita dalla definizione del tipo):
gestiscono la *logica di dominio* che va oltre la struttura.

---

## Funzioni custom

Dove gli hook **reagiscono** a eventi standard, le funzioni custom **aggiungono operazioni nuove**,
esposte come endpoint dedicati.

Si definiscono come funzioni pubbliche nel modulo dell'Arke e si chiamano via API:

| path | dispatch |
|---|---|
| `GET\|POST /:arke_id/function/:function_name` | `call_func(arke, :function_name, [arke])` |
| `GET\|POST /:arke_id/unit/:unit_id/function/:function_name` | `call_func(arke, :function_name, [arke, unit])` |
| `GET\|POST /group/:group_id/function/:function_name` | funzione a livello di gruppo |

La `conn` (parametri, body, member autenticato) viene iniettata nell'Arke via `runtime_data`, così
la funzione può accedervi. La funzione **non restituisce una `conn`**: restituisce un dato, e il
controller lo mappa nella risposta HTTP — `{:ok, content, status}`, `{:error, msg, status}`,
`{:file, binary, filename}`, oppure un valore qualsiasi (→ `200`).

```elixir
defmodule MyProject.Arke.Ordine do
  use Arke.System
  arke id: "ordine" do end

  # POST /ordine/unit/:unit_id/function/conferma
  def conferma(_arke, unit) do
    {:ok, unit} = Arke.QueryManager.update(unit, stato: "confermato")
    {:ok, Arke.StructManager.encode(unit, type: :json), 200}
  end
end
```

Così il sistema non offre solo CRUD generico, ma il **vocabolario specifico del dominio**: un
`Ordine` "conferma", un `Documento` "pubblica" o "archivia".

---

## Hook vs Funzioni custom

| | Hook | Funzione custom |
|---|---|---|
| innesco | un evento del ciclo di vita | una chiamata esplicita all'endpoint |
| scopo | logica trasversale al CRUD | un'operazione di dominio nuova |
| endpoint | gli stessi standard | `/:arke_id/function/:name` |
| dichiarazione | override di `on_create/...` | funzione pubblica sull'Arke |

Le due cose si compongono: una funzione custom può scatenare un hook, un hook può applicare i
permessi del member corrente. La coerenza del modello rende queste combinazioni prevedibili.

---

*Vedi anche: [Query Manager](./02_query_manager.md) · [Managers](./03_managers.md) ·
[Autenticazione](./05_autenticazione.md) · [Database](./01_database.md)*
