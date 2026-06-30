<!-- discorso: ../discorsi/07_registry.md · ritmo: veloce -->

# Il Registry: la struttura è dichiarativa

File JSON nei pacchetti → si **dichiara** e si fa il **seed**

```bash
mix arke.export_data --project my_project --splitfile

mix arke.seed_project --project my_project
```

> Il registry è **versionato in git** — non uno stato creato a mano.
