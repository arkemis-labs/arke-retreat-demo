# Arke — Rappresentazione su Database

L'idea chiave da tenere a mente leggendo questo documento:

> **Tipo e istanza vivono nella stessa tabella.** Un Arke, un Parameter, una Unit, un Group,
> un Project — sono tutti righe di `arke_unit`. Cambia solo il valore della colonna `arke_id`,
> che dice di *che cosa* quella riga è un'istanza.

---

# Arke

Un Arke è la **definizione di un tipo**. Su database è rappresentato da una riga della tabella
`arke_unit`, il cui `id` è in genere il nome dell'arke (es. `frutto`).

Trattandosi a sua volta di una Unit del meta-arke `arke`, la sua colonna `arke_id` vale `arke`.

`arke_unit` contiene le seguenti colonne:

| colonna | tipo | note |
|---|---|---|
| `id` | `string` (PK) | identificativo univoco, spesso coincide col nome |
| `arke_id` | `string` | il tipo della riga: per un Arke vale `arke` |
| `data` | `jsonb` (map) | valori dei parameter con `persistence: "arke_parameter"` |
| `metadata` | `jsonb` (map) | dati di servizio (es. il `project`) ed eventuali override |
| `inserted_at` | `timestamp` | creazione |
| `updated_at` | `timestamp` | ultima modifica |

Esempio di Arke nella `arke_unit`:

| id | arke_id | data | metadata | inserted_at | updated_at |
|---|---|---|---|---|---|
| `frutto` | `arke` | `{"label": "Frutto", "type": "arke", "active": true}` | `{"project": "my_project"}` | ... | ... |

Gli attributi scalari dell'Arke (`label`, `type`, `active`, …) stanno in `data`. I suoi
**parameter non stanno qui**: sono righe a sé, collegate tramite `arke_link` (vedi
[Associazione ad arke](#associazione-ad-arke)).

### API e accesso

Su questi dati si opera con **`Arke.QueryManager`** (CRUD + query); la definizione dell'Arke,
caricata in memoria, si recupera da **`Arke.Boundary.ArkeManager`**. Lato HTTP, **`arke_server`**
espone automaticamente ogni Arke tramite endpoint REST standard.

→ Vedi [Query Manager](./02_query_manager.md), [Managers](./03_managers.md) e
[Codice → Router](./04_codice.md#il-router).

---

# Parameter

Un Parameter è un **campo riutilizzabile**. Anch'esso è una riga di `arke_unit`, il cui `id` è
in genere il nome del parametro. La particolarità: il suo `arke_id` **è il tipo del valore**.

Esempio di Parameter nella `arke_unit`:

| id | arke_id | data | metadata | inserted_at | updated_at |
|---|---|---|---|---|---|
| `peso` | `integer` | `{"label": "Peso", "min": 0, "required": false}` | `{"project": "my_project"}` | ... | ... |

In `data` vive la configurazione del parametro: vincoli, valore di default, flag. Quali chiavi
siano ammesse dipende dal tipo (vedi sotto).

## Tipi di parameter

Il Parameter è una Unit il cui Arke è il **tipo del valore**. I tipi disponibili a sistema:

| tipo (`arke_id`) | rappresenta | attributi di vincolo principali |
|---|---|---|
| `string` | testo | `min_length`, `max_length`, `values`, `multiple`, `unique`, `strip`, `lowercase` |
| `integer` | intero | `min`, `max`, `values`, `multiple`, `unique` |
| `float` | decimale | `min`, `max`, `values`, `multiple` |
| `boolean` | vero/falso | `default_boolean` |
| `date` | data (`YYYY-MM-DD`) | `default_date` |
| `time` | orario (`HH:MM:SS`) | `default_time` |
| `datetime` | data+ora | `default_datetime` |
| `dict` | mappa chiave/valore | `default_dict` |
| `list` | lista | `default_list` |
| `link` | riferimento ad altra Unit | `arke_or_group_id`, `connection_type`, `direction`, `depth`, `multiple` |
| `dynamic` | valore calcolato a runtime | — |
| `binary` | dati binari | — |

Alcuni vincoli sono comuni a (quasi) tutti i tipi: `required`, `nullable`, `default_*`, `label`,
`persistence`.

### `persistence`: dove finisce fisicamente il valore

Ogni parameter dichiara una `persistence` che decide **dove** il suo valore viene salvato su una Unit:

- **`table_column`** → è una colonna reale di `arke_unit`. Sono solo: `id`, `arke_id`,
  `metadata`, `inserted_at`, `updated_at`.
- **`arke_parameter`** → il valore vive **dentro la mappa `data`** (jsonb). È il caso di tutti
  gli altri parameter.

È questo il meccanismo che permette di avere uno schema fisso (poche colonne) ma una struttura
dati arbitraria per tipo: la flessibilità del meta-modello sta nel jsonb di `data`.

## Associazione ad arke

Perché un Arke abbia determinati parameter, essi devono essere **linkati**. Questo significa
creare una riga nella tabella `arke_link`, con `type: "parameter"`, dove l'Arke è il `parent`
e il Parameter è il `child`:

| type | parent_id | child_id | metadata |
|---|---|---|---|
| `parameter` | `frutto` | `peso` | `{}` |
| `parameter` | `frutto` | `nome` | `{"required": true}` |

La direzione è `child` (l'Arke "possiede" i suoi parameter come figli). È la relazione
**strutturale 1:N** descritta in topologia: lo stesso parameter `peso` può essere figlio di più
Arke diversi, riusando lo stesso vocabolario.

## Override in metadata

Il `metadata` del link Arke–Parameter permette l'**override dei vincoli del parameter** solo nel
contesto di quell'Arke. È il secondo dei due livelli di validazione:

- **Sul Parameter** → vincoli intrinseci, validi ovunque (es. `min`/`max` di `peso`).
- **Sull'associazione** → vincoli che valgono solo per quell'Arke, dichiarati nel `metadata` del link.

Così lo stesso campo `descrizione` può essere facoltativo in un Arke e obbligatorio in un altro,
o avere un default diverso, senza essere ridefinito:

| type | parent_id | child_id | metadata |
|---|---|---|---|
| `parameter` | `frutto` | `peso` | `{"required": true, "default_integer": 0}` |

Le chiavi ammesse nel `metadata` sono le stesse del tipo del parameter (`required`, `nullable`,
`default_*`, `label`, `min`, `max`, …).

---

# Unit

Una Unit è un'**istanza concreta** di un Arke. Lato DB si trova anch'essa nella `arke_unit`,
avendo come `arke_id` l'`id` del suo Arke (es. `frutto`).

I suoi valori finiscono dove la `persistence` del relativo parameter comanda: quelli
`arke_parameter` dentro `data`, quelli `table_column` nelle colonne omonime.

| id | arke_id | data | metadata | inserted_at | updated_at |
|---|---|---|---|---|---|
| `mela` | `frutto` | `{"nome": "Mela", "peso": 150}` | `{"project": "my_project"}` | ... | ... |

La conformità dei valori in `data` con la definizione dell'Arke è garantita automaticamente in
fase di scrittura: non serve validazione esplicita per ogni entità.

### API e accesso

Le Unit si creano, leggono e filtrano con **`Arke.QueryManager`** (`create`, `get_by`,
`filter_by`, query componibili e operatori di confronto).

→ Vedi [Query Manager](./02_query_manager.md).

---

# Link

Un Link è una **relazione persistita tra due Unit**. È salvato nella tabella `arke_link`.

A differenza di `arke_unit`, `arke_link` **non ha `id` né timestamp**: la sua chiave primaria è
la coppia `(parent_id, child_id)`. Colonne:

| colonna | tipo | note |
|---|---|---|
| `type` | `string` (default `"link"`) | qualifica la semantica della relazione |
| `parent_id` | `string` → `arke_unit.id` | estremo "padre"; FK con `ON DELETE CASCADE`; parte della PK |
| `child_id` | `string` → `arke_unit.id` | estremo "figlio"; FK con `ON DELETE CASCADE`; parte della PK |
| `metadata` | `jsonb` (map) | attributi propri della connessione |

Entrambe le foreign key sono indicizzate (`parent_id`, `child_id`) per rendere veloce la
traversata in entrambe le direzioni. Il `CASCADE` garantisce che, eliminata una Unit, spariscano
anche i suoi link.

Un Link è sempre **direzionato**: va da una Unit `parent` a una Unit `child`. Una connessione
descritta come:

```elixir
%{
  parent_id: "article_001",
  child_id: "tech",
  type: "category"
}
```

su database è semplicemente questa riga:

| type | parent_id | child_id | metadata |
|---|---|---|---|
| `category` | `article_001` | `tech` | `{}` |

### Traversata del grafo

La lettura topologica — partire da una Unit e seguirne i Link per `depth`, `direction`
(`:child` / `:parent`) e `type` — si fa con `Arke.QueryManager.link/3`. Sono i quattro assi della
traversata descritti nella sezione *Topologia*.

→ Vedi [Query Manager → `link`](./02_query_manager.md#le-funzioni).

## Type

La colonna `type` qualifica il **significato** della relazione. Il default è `"link"` (relazione
generica / autonoma). Alcuni valori ricorrenti:

| `type` | uso |
|---|---|
| `link` | Link autonomo generico (relazione N:N, vedi *Topologia*) |
| `parameter` | collega un Arke (`parent`) ai suoi Parameter (`child`) |
| `group` | collega un Group (`parent`) agli Arke che contiene (`child`) |
| *custom* | relazioni di dominio (es. `category`, `author`, …) |

Una traversata può essere limitata a uno o più `type`, ignorando il resto della topologia.

I link "strutturali" generati da un **parameter di tipo `link`** usano come `type` il
`connection_type` dichiarato nel parameter, e vengono creati/aggiornati automaticamente da Arke
quando si scrive il valore del campo sulla Unit.

---

# Group

Un Group **aggrega più Arke** sotto un'unica categoria logica. È una Unit dell'Arke `group`, e ha
quindi sempre `arke_id = group`:

| id | arke_id | data | metadata |
|---|---|---|---|
| `contenuti` | `group` | `{"label": "Contenuti", "arke_list": ["articolo", "video"]}` | `{"project": "my_project"}` |

L'appartenenza al gruppo è materializzata da link con `type: "group"` (il Group è il `parent`,
l'Arke membro è il `child`). Il campo `arke_list` in `data` tiene l'elenco degli Arke del gruppo;
la sincronizzazione coi link è gestita da Arke.

Il Group permette di **regolare i permessi** e di **assegnare custom function** valide per
l'intero insieme di Arke, senza doverle ripetere su ciascuno. A sistema esistono già alcuni
gruppi (es. `parameter`, `arke_or_group`, `arke_auth_member`).

## Arke auth member

Un **member** (`ArkeAuth.Core.Member`) è l'entità autenticata all'interno di un Project.
Appartiene al gruppo di sistema `arke_auth_member`.

Caratteristiche:

- **Riferisce un utente** `arke_system` tramite `data.arke_system_user`: l'identità anagrafica
  vive nel project `arke_system`, il member la contestualizza in un project specifico.
- **Porta i permessi**: i permessi effettivi nascono dalla combinazione di un livello base
  pubblico (`member_public`) e di quelli specifici del member, risolti percorrendo la sua
  gerarchia di link.
- **È per-project**: lo stesso utente può essere member di più Project con ruoli diversi.

È il punto in cui `arke_auth` si innesta sul meta-modello: l'autenticazione non è una tabella a
parte, ma Unit e Link come tutto il resto.

# Project

Il Project è il **contesto di isolamento** (multi-tenancy). Anch'esso è una Unit di `arke_unit`,
dell'Arke `arke_project`:

| id | arke_id | data | metadata |
|---|---|---|---|
| `my_project` | `arke_project` | `{"label": "My Project", "type": "postgres_schema"}` | `{}` |

Su PostgreSQL l'isolamento è implementato **uno schema per Project**: alla creazione di un Project
(`type: "postgres_schema"`) Arke esegue `CREATE SCHEMA "<id>"` e vi replica le tabelle
`arke_unit` / `arke_link` via migration (`prefix: <id>`). Ne consegue che:

- ogni Project ha la propria copia delle due tabelle, nel proprio schema;
- strutture, istanze, relazioni e configurazioni di un Project non interferiscono mai con un altro.

Il multi-tenancy non è un'aggiunta: è una conseguenza diretta di come Arke usa gli schemi di
PostgreSQL.

## arke_system

`arke_system` è il **Project di sistema**: lo schema di default che fa da bootstrap al meta-modello.
Mentre i Project applicativi (`my_project`, …) ospitano i dati di dominio, `arke_system` contiene le
**definizioni base** su cui tutto il resto si appoggia:

- i meta-Arke: `arke`, `parameter`, `group`, `arke_project`, `arke_link`;
- i tipi di parameter (`string`, `integer`, `link`, …);
- i gruppi di sistema (`parameter`, `arke_or_group`, `arke_auth_member`, …);
- le righe dei Project stessi (ogni `arke_project` vive qui) e gli User di `arke_auth`.

Due conseguenze pratiche:

- **Fallback dei manager**: cercando una definizione in un project, i [manager](./03_managers.md)
  ripiegano su `arke_system` se non la trovano localmente — così il meta-modello di sistema è
  visibile ovunque senza essere duplicato;
- **Origine**: il contenuto di `arke_system` non si crea a mano, viene inserito tramite seed dal [Registry](#registry).

---

# Registry

Il **Registry** è la definizione **dichiarativa** del meta-modello: file JSON (uno per tipo —
`arke`, `parameter`, `group`, `link`) che ogni pacchetto Arke porta con sé sotto `lib/registry/`.
Invece di creare Arke e Parameter a mano (via [API](./04_codice.md#il-router) o console), li si
**dichiara** e si fa il seed.

Ogni pacchetto (`arke`, `arke-auth`, …) espone il proprio Registry, diviso in due ambiti:

| ambito | cartella | dove viene caricato | cosa contiene |
|---|---|---|---|
| **system** | `registry/system/` | solo in `arke_system` | il meta-modello base (meta-Arke, tipi di parameter, gruppi di sistema) |
| **shared** | `registry/shared/` | in **ogni** Project | definizioni comuni a tutti i tenant (es. l'Arke `user`, i ruoli `super_admin` / `member_public`) |

Il caricamento avviene con il mix task di seeding, che fonde i Registry del core e di **tutte le
dipendenze** Arke e li scrive sia nei [manager in-memory](./03_managers.md) (ETS) sia sul database
(tabelle del relativo schema):

```bash
mix arke.seed_project --project my_project   # un project → riceve i registry "shared"
mix arke.seed_project --all                  # arke_system ("system") + tutti i project ("shared")
```

In sintesi, il Registry è la **sorgente di verità versionabile** della struttura: `system` →
`arke_system`, `shared` → tutti i Project.

---

# Permessi

Anche i permessi sono **Link**. Un permesso è una riga di [`arke_link`](#link) con
`type: "permission"`:

| campo | valore |
|---|---|
| `parent_id` | il **ruolo**: `member_public` (tutti) oppure l'`arke_id` del [Member](#arke-auth-member) (es. `super_admin`) |
| `child_id` | l'`arke_id` su cui il permesso si applica |
| `metadata` | il dizionario dei permessi (vedi sotto) |

Esempio — il ruolo `installer` può leggere e creare `ordine`, ma non modificarlo né eliminarlo:

| type | parent_id | child_id | metadata |
|---|---|---|---|
| `permission` | `installer` | `ordine` | `{"get": true, "post": true, "put": false, "delete": false}` |

## GET, POST, PUT, DELETE

Il `metadata` porta quattro flag booleani, uno per verbo HTTP / operazione CRUD:

| flag | verbo | operazione |
|---|---|---|
| `get` | `GET` | leggere / elencare le Unit dell'Arke |
| `post` | `POST` | crearne di nuove |
| `put` | `PUT` | aggiornarle |
| `delete` | `DELETE` | eliminarle |

Il [plug `Permission`](./04_codice.md#le-pipeline-plug) traduce il metodo della richiesta nel flag
corrispondente e, se è `false`, risponde `403`. I permessi effettivi nascono dalla **fusione** del
baseline `member_public` con quelli del ruolo del Member. Casi speciali: `super_admin` → tutto
`true`; Member con `subscription_active: false` → tutto `false`.

## Filtri custom

Oltre ai quattro flag, un permesso può portare un **`filter`**: una condizione di query che
restringe **quali** Unit il Member può vedere o toccare — non solo *se* può. È il livello più fine,
la sicurezza per riga.

Il `filter` dichiarato nel `metadata` del permesso viene applicato automaticamente alla query
dell'endpoint (il plug lo assegna come `permission_filter`). Esempi: un installatore vede solo gli
ordini di cui è autore; un utente solo le proprie pratiche. Il flag `child_only` restringe inoltre
alla sola topologia collegata al Member.

I tre livelli di controllo — **Project** (esiste un Member non `inactive`), **Arke** (i flag CRUD),
**Unit** (il `filter`) — sono quelli descritti in [*Feature → Visibilità e Permessi*](../filosofia/03_feature.md#visibilità-e-permessi).

---

*Vedi anche: [Overview](./00_overview.md) · [Query Manager](./02_query_manager.md) ·
[Managers](./03_managers.md) · [Codice](./04_codice.md) · [Autenticazione](./05_autenticazione.md)*
