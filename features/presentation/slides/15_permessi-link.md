<!-- discorso: ../discorsi/15_permessi-link.md · ritmo: medio -->

# Anche i permessi sono Link

`arke_link` con `type = permission` · **parent** = ruolo · **child** = Arke

| ruolo → arke           | get | post | put | delete |
| ---------------------- | :-: | :--: | :-: | :----: |
| `installer` → `ordine` |  ✓  |  ✓   |  ✗  |   ✗    |

- PUT/DELETE a `false` -> **403**
- `super_admin` → tutto `true`
- **`filter`**: non solo *se* puoi, ma *su quali righe* (es. solo i propri ordini)