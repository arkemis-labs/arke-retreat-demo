# Arke — Introduzione e Filosofia

---

## Il Problema

Ogni volta che si costruisce un'applicazione, si ripete sempre lo stesso ciclo:

- Si definisce una struttura dati (uno schema, un modello)
- Si scrive la logica per gestirla (CRUD, validazione, relazioni)
- Si espone un'API
- Si costruisce un'interfaccia per interagirci

Questo ciclo si ripete per ogni entità, per ogni progetto, per ogni cliente.
Il risultato è codice scritto a mano, rigido per costruzione, difficile da evolvere senza intervento tecnico.

**Il problema non è la complessità tecnica — è la rigidità strutturale.**

---

## La Domanda

> E se il dato potesse descrivere se stesso?
> E se la struttura di un'informazione potesse essere definita senza dover riscrivere il sistema?

Questa è la domanda da cui nasce Arke.

---

## Cos'è Arke

Arke è un **framework meta-modello**: un sistema che non gestisce dati specifici, ma gestisce la *definizione* dei dati.

La distinzione fondamentale è tra il *tipo* e l'*istanza*:

- Il **tipo** definisce cos'è qualcosa — la sua forma, i suoi campi, le sue regole
- L'**istanza** è qualcosa — un'occorrenza concreta di quel tipo

È la stessa distinzione tra "cos'è un Prodotto" e "questo prodotto specifico".
Ma in Arke questa distinzione è esplicita, gestita dal sistema, e modificabile dinamicamente.

Non si tratta di un CMS, non è un ORM, non è un generatore di codice.
È un **layer di astrazione sul dato**: un sistema che tratta la struttura dell'informazione come un cittadino di prima classe — modificabile, interrogabile, componibile.

---

## Perché Esiste

### Il dato è statico, il business non lo è

I modelli tradizionali fissano la struttura dei dati nel codice. Quando il business evolve — e lo fa sempre — servono sviluppatori per aggiornare schemi, migrazioni, API e interfacce.

Arke separa la *struttura del dato* dalla *logica del sistema*. La struttura può cambiare senza toccare il sistema.

### Ogni cliente è diverso, il sistema no

In contesti multi-tenant, ogni cliente spesso ha esigenze leggermente diverse. Con un approccio tradizionale, questo porta a fork del codice, configurazioni complesse, o compromessi.

Con Arke, ogni contesto isolato può avere la propria struttura dati, i propri campi, le proprie relazioni — senza duplicare l'infrastruttura.

### Il tempo di sviluppo è un costo

Definire modelli, generare API, costruire interfacce di gestione: tutto questo richiede tempo. Arke riduce questo ciclo portando fuori dal codice ciò che può essere configurazione.

---

## Cosa Abilita

- **Modellazione dinamica**: la struttura dei dati può essere definita e modificata senza deploy
- **API automatiche**: ogni tipo di dato definito genera automaticamente endpoint per gestire le sue istanze
- **Multi-tenancy nativa**: ogni contesto è isolato, con la propria struttura e i propri dati
- **Interfaccia amministrativa**: la Console permette di gestire tipi, istanze e relazioni visualmente
- **Comportamenti personalizzabili**: ogni tipo di dato può avere logica specifica nel proprio ciclo di vita

---

## A Chi Si Rivolge

- **Team di prodotto** che costruiscono piattaforme configurabili o multi-tenant
- **System integrator** che devono adattare la stessa base a clienti con esigenze diverse
- **Startup** che vogliono muoversi velocemente su modelli di dati che cambiano spesso
- **Progetti interni** dove la velocità di iterazione conta più della customizzazione low-level

---

## La Visione

L'obiettivo è spostare il lavoro di modellazione dall'ingegneria alla configurazione — senza perdere il controllo, la validazione, o la coerenza.

> Costruire sistemi che si adattano al dominio, non domini che si adattano al sistema.

---

*Continua in: [02 — Architettura e Definizione](./02_architettura.md)*
