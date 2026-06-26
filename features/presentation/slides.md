# Presentazione `features/` — Lista delle slide

> Outline della presentazione (solo lista delle slide, **non** il contenuto).
> Il contenuto di ogni slide verrà scritto in markdown in un secondo momento.

## Contesto e strategia

Questa presentazione viene **dopo** quella del collega, basata sui file in `filosofia/`.
Quella prima metà è tutta concetti, zero codice: quando iniziamo noi, il pubblico ha già capito
*cosa* è Arke e *perché* esiste — il meta-modello, i building block (Arke, Unit, Parameter, Link,
Group, Project), i due modi di leggere i dati, cosa sono autenticazione/permessi/hook/funzioni
custom come idee, e la filosofia della persistenza (ETS, libcluster).

La nostra metà è la **macchina** — *come* tutto questo è realmente costruito. Il filo conduttore
più forte del nostro materiale è:

> Tutto ciò che il pubblico ha appena sentito è, in concreto, **righe in due tabelle**
> (`arke_unit` e `arke_link`), popolate in modo dichiarativo ed esposte da **un solo router
> dinamico**.

Strategia: passare veloci su ciò che il collega ha già spiegato come concetto, e rallentare sulle
rivelazioni concrete che sono effettivamente nuove.

**Nota importante:** la sezione "ecosistema dei 4 moduli" esiste nei file `filosofia/` ma è
**commentata**, quindi il pubblico non l'avrà vista. Per noi è materiale nuovo.

---

## Le slide

### Atto 0 — Ponte (1 slide)

1. **Dal concetto alla macchina.** Breve ponte che richiama le sei idee del collega e re-inquadra
   il resto del talk come "ora vediamo come è davvero implementato". Niente ri-spiegazioni: serve
   solo a dare il frame. *(Veloce.)*

### Atto 1 — L'ecosistema (1 slide) · fonte: `00_overview.md`

2. **Le quattro repo e come dipendono tra loro.** `arke` (core), `arke-postgres` (persistenza),
   `arke-auth` (identità), `arke-server` (API HTTP) *(Veloce.)*

### Atto 2 — "Tutto è righe in due tabelle" · fonte: `01_database.md`
*Questo atto è il cuore del valore della nostra presentazione.*

3. **La rivelazione: la tabella `arke_unit`.** L'idea chiave: tipo e istanza vivono nella *stessa*
   tabella, distinti solo dalla colonna `arke_id`. Un Arke, un Parameter, una Unit, un Group, un
   Project — tutte righe. *(Soffermarsi.)*

4. **Come si salva un Parameter, e il trucco di `persistence`.** L'`arke_id` di un Parameter *è* il
   tipo del suo valore. L'impostazione `persistence` (`table_column` vs. il blob JSONB `data`) è il
   meccanismo che permette a uno schema fisso di contenere strutture dati arbitrarie e diverse per
   tipo. *(Soffermarsi.)*

5. **`arke_link`: una tabella per ogni tipo di relazione.** Chiave primaria `(parent_id, child_id)`,
   cascade delete, e la colonna `type` che unifica tutto: link strutturali da parameter, membership
   di gruppo, link autonomi N:N e permessi sono tutti righe qui con `type` diverso. **Una delle
   slide più forti.** *(Soffermarsi.)*

6. **Isolamento dei tenant e bootstrap di sistema.** Ogni Project ha il proprio schema PostgreSQL
   (`CREATE SCHEMA`), e `arke_system` è il project di bootstrap su cui i manager fanno fallback. Il
   pubblico ha sentito il multi-tenancy come *concetto*; qui c'è il meccanismo reale. *(Veloce.)*

7. **Il Registry: dichiarare il meta-modello.** Il meta-modello è definito in modo dichiarativo in
   file JSON e caricato via un task di seed (con ambiti `system` vs. `shared`). **Completamente
   nuovo** — non c'era nella filosofia. *(Veloce.)*

### Atto 3 — Il layer d'accesso · fonti: `02_query_manager.md`, `03_managers.md`

8. **QueryManager: una sola API per tutto.** Un unico punto d'accesso sia per il filtraggio
   relazionale sia per la traversata del grafo (`link/3`), con query componibili. *(Medio.)*

9. **I limiti del QueryManager.** I tradeoff onesti: niente operazioni bulk, niente select di
   colonne parziali, gli hook del ciclo di vita scattano sempre (e può essere lento), e bisogna
   leggere una Unit prima di aggiornarla o eliminarla. Una slide franca "ecco quanto costa" funziona
   bene con un pubblico tecnico ed è genuinamente interessante, non tediosa. *(Veloce.)*

10. **Managers ed ETS.** I registri in-memory concreti (Arke, Parameter, Group) e i manager
    operativi (Link, Struct, File). Il collega ha già coperto la *filosofia* di ETS, quindi qui è
    soprattutto un veloce "ecco i moduli veri". *(Veloce.)*

### Atto 4 — Dove vive il codice · fonte: `04_codice.md`

11. **Il router dinamico su `:arke_id`.** Un solo set di route serve *ogni* Arke esistente, senza
    controller per tipo — più la pipeline di richiesta (i plug `GetProject`, `Auth`, `Permission`).
    È il payoff concreto della promessa "API automatica". *(Soffermarsi.)*

12. **Funzioni custom.** Come aggiungere operazioni di dominio oltre il CRUD, e come si distinguono
    dagli hook. *(Medio)*

13. **Hook del ciclo di vita.** La sequenza di esecuzione attorno a create/update/delete, gli hook a
    livello di gruppo, e come l'override di un hook cambia il comportamento di un endpoint senza
    toccare il router. Il concetto è già stato coperto, quindi mostrare la sequenza e proseguire.
    *(Veloce.)*

### Atto 5 — Identità e accessi · fonti: `05_autenticazione.md` + permessi da `01_database.md`

14. **JWT.** Dire che usiamo JWT. Il pubblico ha già il concetto, quindi tenerla rapida. *(Veloce.)*

15. **I permessi sono Link.** Un permesso è una riga `arke_link` con `type: "permission"`, che porta
    i flag CRUD (GET/POST/PUT/DELETE) e un `filter` opzionale a livello di riga. Il concetto è già
    stato coperto; la parte nuova è l'implementazione basata su link. *(Medio.)*

### Atto 6 — Chiusura (1 slide)

16. **Sintesi: concetto → implementazione.** Mappare ogni idea della filosofia sulla macchina
    concreta (le due tabelle, il registry, il router dinamico). Lega insieme le due metà del
    retreat. *(Veloce.)*

---

## Come evitare che diventi pesante

- **16 slide core.** Circa cinque sono recap veloci di cose già coperte dal collega; quattro
  meritano tempo vero.
- **Le slide che giustificano la nostra metà:** la 5 (una tabella per tutte le relazioni), la 6 (uno
  schema per tenant), la 9 (i limiti onesti) e la 11 (il router dinamico).
- **Tenere FUORI dalle slide** — è qui che diventa tedioso: la tabella completa degli operatori di
  confronto, l'elenco completo degli endpoint, ogni nome di hook, ogni tipo di parameter. Dire che
  esistono, mostrarne due o tre esempi, e andare avanti.

## Aggiunte opzionali

- **Demo live in `iex`** dopo la slide 11: creare un parameter, un Arke e un link, poi colpire
  subito l'endpoint auto-generato. Alto coinvolgimento e il miglior antidoto a un tratto arido.
- Se si è corti di tempo, accorpare la slide 13 nella 12.
