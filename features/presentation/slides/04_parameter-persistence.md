<!-- discorso: ../discorsi/04_parameter-persistence.md · ritmo: soffermarsi -->

# Parametri

Anche un parametro è una riga. Il suo `arke_id` **è il tipo del valore**.

| id     | arke_id   | data                              |
| ------ | --------- | --------------------------------- |
| `peso` | `integer` | `{label: "Peso", min: 0}`         |
| `nome` | `string`  | `{label: "Nome", required: true}` |

- Schema fisso, pochissime colonne → la flessibilità vive nel **JSONB `data`**
- Qualsiasi tipo
- Stesso campo riusabile su più Arke, con vincoli sovrascrivibili per Arke nella `arke_link`
