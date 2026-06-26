<!-- discorso: ../discorsi/04_parameter-persistence.md · ritmo: soffermarsi -->

# Il Parameter: il tipo è nell'`arke_id`

Anche un campo è una riga. Il suo `arke_id` **è il tipo del valore**.

| id | arke_id | data |
|---|---|---|
| `peso` | `integer` | `{label: "Peso", min: 0}` |
| `nome` | `string` | `{label: "Nome", required: true}` |

- Schema fisso, pochissime colonne → la flessibilità vive nel **JSONB `data`**
- Qualsiasi campo, di qualsiasi tipo — **senza una migration**
- Stesso campo riusabile su più Arke, con vincoli sovrascrivibili per Arke
