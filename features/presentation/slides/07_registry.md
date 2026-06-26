<!-- discorso: ../discorsi/07_registry.md · ritmo: veloce -->

# Il Registry: la struttura è dichiarativa

File JSON nei pacchetti → si **dichiara** e si fa il **seed** (niente creazione a mano).

| ambito | caricato in | contiene |
|---|---|---|
| `system` | solo `arke_system` | il meta-modello di base |
| `shared` | ogni project | definizioni comuni a tutti i tenant |

```bash
mix arke.seed_project --project my_project
```

> La struttura del sistema è **versionata in git** — non uno stato creato a mano.
