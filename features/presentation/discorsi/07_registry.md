# Slide 7 — Il Registry

*Ritmo: Veloce. Testo da dire; le slide verranno adattate dopo.*

---

Domanda: tutte queste righe — i meta-arke, i tipi, i gruppi di sistema — chi le mette nel database la prima volta? Non a mano, di sicuro.

La risposta è il Registry. È la definizione dichiarativa del meta-modello: dei file JSON che ogni pacchetto di Arke si porta dietro. Invece di creare Arke e Parameter a mano, li si dichiara in un file e si fa il seed.

Ci sono due ambiti. Il `system`, che viene caricato solo in `arke_system`: è il meta-modello di base. E lo `shared`, che viene caricato in ogni project: le definizioni comuni a tutti i tenant — per esempio l'Arke "user", i ruoli di base. -> da chiarire di cosa si parla

Si lancia un mix task, `arke.seed_project`, che fonde i registry del core e di tutte le dipendenze e li scrive sia in memoria sia sul database.

Il punto da portare a casa: la struttura del sistema è versionabile. Sta nei file, in git. Non è uno stato che qualcuno ha creato cliccando da qualche parte.
