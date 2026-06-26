<!-- discorso: ../discorsi/03_arke-unit.md · ritmo: soffermarsi -->

# Tutto è una riga di `arke_unit`

**Tipo e istanza vivono nella stessa tabella.**
Li distingue una sola colonna: `arke_id`.

| id | arke_id | data |
|---|---|---|
| `frutto` | `arke` | `{label: "Frutto", …}` |
| `mela` | `frutto` | `{nome: "Mela", peso: 150}` |

- `frutto` è la **definizione del tipo** → `arke_id = arke`
- `mela` è l'**istanza** → `arke_id = frutto`

> Il modello è un dato: lo crei, lo modifichi, lo cancelli come ogni riga.
