# Arke — Autenticazione e Permessi

L'autenticazione in Arke è il modulo **`arke_auth`**. Non è un layer esterno: utenti, member e
permessi sono Unit e Link come qualsiasi altro dato (vedi [Database](./01_database.md)). Cambia solo
il fatto che il sistema li riconosce come **soggetti** delle operazioni, non solo come oggetti.

---

## I due soggetti: User e Member

La distinzione centrale di `arke_auth`:

| | **User** | **Member** |
|---|---|---|
| arke | `user` | un arke del gruppo `arke_auth_member` (es. `super_admin`) |
| dove vive | nel project **`arke_system`** (globale) | nel **project applicativo** (per-tenant) |
| cosa contiene | credenziali: `username`, `password_hash` | ruolo, permessi, dati nel contesto del project |
| collegamento | — | `data.arke_system_user` → l'`id` dello User |
| molteplicità | uno | **N** — un Member per ogni project a cui l'utente partecipa |

In una frase: **lo User si autentica, il Member autorizza**. La stessa persona è un solo User in
`arke_system`, ma diventa un Member distinto in ciascun project, con ruolo e permessi propri.

Lo **User** è definito con `use Arke.System` e gestisce la password in modo sicuro:

- in `before_load` (alla creazione) la password viene cifrata con **bcrypt** e salvata come
  `password_hash`; il campo `password` in chiaro non viene mai persistito;
- in `before_struct_encode` il `password_hash` viene rimosso dalla serializzazione: l'API non lo
  restituisce mai.

---

## Il flusso di login

`ArkeAuth.Core.Auth.validate_credentials(username, password, project)`:

```
1. email_password_auth  → cerca lo User (in :arke_system) per username
                          e verifica la password con bcrypt (checkpw su password_hash)
2. get_project_member   → trova il Member nel `project` con arke_system_user == user.id
                          (gruppo "arke_auth_member"); rifiuta se `inactive`
3. create_tokens        → genera access_token + refresh_token per quel Member
```

Risultato: `{:ok, member, access_token, refresh_token}`. Se manca lo User, la password è errata o
non esiste un Member nel project → `{:error, "unauthorized"}`.

> Nota di sicurezza: se lo username non esiste, il codice esegue comunque un `dummy_checkpw()` per
> non rivelare — dai tempi di risposta — se l'utente è presente.

---

## Token JWT

Arke usa **Guardian** per emettere JSON Web Token. `create_tokens/2` produce due token:

| token | scopo | TTL (default) |
|---|---|---|
| **access_token** | autentica ogni richiesta (`Authorization: Bearer …`) | 1 settimana |
| **refresh_token** | ottenere un nuovo access_token senza ri-login | 4 settimane |

Cosa c'è dentro al token — `subject_for_token/2` mette nel `sub`:

```elixir
%{id: member.id, project: member.metadata.project}  # + i dati del member
```

In entrata, `resource_from_claims/1` rilegge `id` e `project` dal `sub`, ricarica il Member da quel
project (gruppo `arke_auth_member`) e verifica che non sia `inactive`. È il Member, non lo User, la
**resource** associata a una richiesta autenticata.

**Refresh** — `refresh_tokens/2` verifica il refresh_token, lo scambia per un nuovo access_token
(`Guardian.exchange`) e rigenera un refresh_token.

---

## Password

Tutta la gestione passa da bcrypt (`Comeonin.Bcrypt`):

- `User.check_password/2` — verifica una password contro l'hash;
- `User.update_password/2` — ricifra e salva il nuovo hash;
- `Auth.change_password/3` — verifica la vecchia password, poi imposta la nuova.

Reset/recupero password e codici OTP monouso sono gestiti da moduli dedicati
(`ResetPasswordToken`, `Otp` / `OtpManager`) e dai relativi endpoint
([vedi sotto](#endpoint)).

---

## Endpoint

Gli endpoint di autenticazione sono fissi (non dinamici su `:arke_id`) e vivono sotto `/lib/auth`
(elenco completo nella tabella *Sistema* del [router](./04_codice.md#il-router)):

| metodo | path | scopo |
|---|---|---|
| `GET` `POST` | `/auth/signin` | login (username + password) |
| `POST` | `/auth/:arke_id/signup` | registrazione di un nuovo Member |
| `POST` | `/auth/refresh` | scambia il refresh_token per un nuovo access_token |
| `POST` | `/auth/verify` | verifica la validità di un token |
| `POST` | `/auth/change_password` | cambio password (autenticato) |
| `POST` | `/auth/recover_password` · `/auth/reset_password[/:token]` | recupero / reset password |

---

## SSO / OAuth (cenni)

Oltre al login username/password, `arke_auth` supporta l'accesso via provider esterni:

- un **`SSOGuardian`** separato emette i token per il flusso SSO;
- i provider configurati vivono nel gruppo `oauth_provider`;
- `POST /auth/signin/:provider` gestisce il login dal client, `POST /auth/:member/:provider` crea il
  Member a partire dall'identità del provider.

---

*Vedi anche: [Codice → Router](./04_codice.md#il-router) per gli endpoint e le pipeline,
[Database → Arke auth member](./01_database.md#arke-auth-member) per la riga del Member.*
