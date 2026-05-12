# Arke — Demo: Filosofia e Visione

Una guida ad alto livello per capire cos'è Arke, perché esiste e come è costruito.
Nessun esempio di codice. Solo concetti, principi, e architettura.

---

## Documenti

### [01 — Introduzione e Filosofia](./01_intro.md)
Il problema che Arke risolve, la domanda da cui nasce, la visione che lo guida.
Cos'è il meta-modello, perché la rigidità strutturale è il vero costo, a chi si rivolge.

### [02 — Architettura e Definizione](./02_architettura.md)
I building block del sistema: Arke, Unit, Parameter, Link, Group, Project.
Come si tengono insieme e come si articola l'ecosistema di moduli.

### [02b — Topologia e Interrogazione](./02b_topologia.md)
I due modelli di lettura dei dati: relazionale e a grafo.
La distinzione tra relazioni strutturali (parameter di tipo link) e Link autonomi (N:N in topologia).
Direzione e semantica parent-child: come ogni Link ha un verso e si percorre in entrambe le direzioni.
Come le entità connesse formano un grafo navigabile.

### [03 — Feature](./03_feature.md)
Le capacità operative del sistema oltre la modellazione:
autenticazione integrata, controllo degli accessi e visibilità, hook sul ciclo di vita degli Arke, funzioni custom per estendere il vocabolario del dominio.

### [04 — Persistenza](./04_persistenza.md)
La gestione dello stato: dal connettore delegato (arke-postgres) alla memoria veloce in ETS,
fino al coordinamento multi-nodo con libcluster per sistemi distribuiti e scalabili.

---

## Filo Narrativo

```
Problema → Meta-modello → Building Block → Topologia → Feature → Persistenza
```

Ogni documento è autonomo ma pensato per essere presentato in sequenza.
Il punto di arrivo naturale della demo è la sezione Persistenza: è lì che la filosofia
incontra l'architettura reale di un sistema pronto per la produzione.

---

## Concetti Chiave

| Termine | Definizione rapida |
|---|---|
| **Arke** | Il tipo — definisce la struttura di un'entità |
| **Unit** | L'istanza — una realizzazione concreta di un Arke |
| **Parameter** | Il campo — una proprietà riutilizzabile tra più Arke |
| **Link strutturale** | Parameter di tipo link — relazione embedded nella definizione dell'Arke (1:N) |
| **Link autonomo** | Connessione topologica indipendente tra Unit, non appartenente a nessun Arke (N:N) |
| **Parent / Child** | I due estremi di un Link — ogni relazione ha un verso canonico e può essere percorsa in entrambe le direzioni |
| **Group** | L'aggregazione — categoria logica che raccoglie più Arke |
| **Project** | Il contesto — unità di isolamento per il multi-tenancy |
| **Hook** | Logica di dominio agganciata al ciclo di vita di un Arke |
| **ETS** | Memoria veloce in-process per dati acceduti frequentemente |
| **libcluster** | Coordinamento automatico tra nodi in ambienti distribuiti |
