<!-- discorso: ../discorsi/12_hook.md · ritmo: veloce -->

# Lifecycle Hook

Callback sul lifecycle di un Arke — una per ogni momento:
validate · create · update · delete.

**Sequenza (create):**

```
load → validate → before_create → persist → on_create
```

- `before_*` → arricchisci o blocca
- `on_*` → effetti collaterali (es. invia mail)
- variante **di gruppo** → logica condivisa da più Arke

> ⚠️ Scattano a **ogni** operazione. Potenti — ma attenti ai loop
> (es. un `on_update` che ne scatena un altro).
