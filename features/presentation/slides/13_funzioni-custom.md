<!-- discorso: ../discorsi/13_funzioni-custom.md · ritmo: medio -->

# Funzioni custom: il vocabolario del dominio

Gli hook **reagiscono**; le funzioni custom **aggiungono** operazioni nuove.

- Funzione pubblica nel modulo dell'Arke → endpoint automatico:
  `POST /:arke_id/function/:nome` (o sulla singola Unit)
- Ricevi `params`, `body`, `member` (iniettati)

```elixir
def conferma(_arke, unit) do
  {:ok, unit} = QueryManager.update(unit, stato: "confermato")
  {:ok, encode(unit), 200}
end
```

> Un ordine "conferma", un documento "pubblica". Oltre il CRUD.
