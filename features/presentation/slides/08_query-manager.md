<!-- discorso: ../discorsi/08_query-manager.md · ritmo: medio -->

# QueryManager: una sola API

Punto d'accesso unico per leggere e scrivere Unit.
**Costruisce** la query, **delega** l'esecuzione al connettore → indipendente dal DB.

**Due modi di leggere:**
- relazionale → `filter_by(arke: :frutto, peso__gte: 100)`
- topologico → `link(unit, depth: 2, direction: :child, type: "ordine")`

**Componibile** — ogni funzione torna una query; niente tocca il DB fino all'esecutore.

> `create` / `update`: validazione + hook, in una sola chiamata.
