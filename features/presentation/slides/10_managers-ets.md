<!-- discorso: ../discorsi/10_managers-ets.md · ritmo: veloce — (slide in forse) -->

# Managers & ETS

**Registri in memoria (ETS)** — i dati più letti, senza colpire il DB ogni volta:
`ArkeManager` · `ParameterManager` · `GroupManager`

**Manager operativi:**
- `QueryManager` — CRUD + query
- `LinkManager` — muta `arke_link`
- `StructManager` — Unit ⇄ JSON (le risposte API)
- `FileManager` — cache degli URL firmati

> Dati caldi in memoria, un manager per ogni responsabilità.
