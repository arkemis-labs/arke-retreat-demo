# Slide 6 — Isolamento dei tenant e bootstrap

*Ritmo: Veloce. Testo da dire; le slide verranno adattate dopo.*

---

Due parole sul multi-tenancy, perché il concetto lo avete già: ogni Project è un contesto isolato. Quello che forse non sapete è come.

Anche il Project è una Unit, ovviamente. Ma su PostgreSQL l'isolamento è fatto con uno schema per Project. Quando create un project, Arke esegue letteralmente un `CREATE SCHEMA` e replica lì dentro le due tabelle, `arke_unit` e `arke_link`. Ogni tenant ha la sua copia delle due tabelle, nel suo schema. Non si toccano mai.

C'è poi un project speciale, `arke_system`: è lo schema di bootstrap. Contiene le definizioni di base su cui si regge tutto il resto — i meta-arke, i tipi di parametro, i gruppi di sistema. E quando un manager cerca una definizione in un project e non la trova, ripiega su `arke_system`. Così il meta-modello di base è visibile ovunque senza essere duplicato in ogni schema.

In una frase: il multi-tenancy non è un'aggiunta, è una conseguenza diretta di come Arke usa gli schemi di Postgres.
