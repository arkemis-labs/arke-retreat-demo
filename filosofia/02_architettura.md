# Arke — Architettura e Definizione

---

## I Building Block

Arke è costruito attorno a un insieme di concetti fondamentali che, combinati, permettono di modellare qualsiasi dominio senza scrivere codice specifico per ogni entità.

---

### Arke — Il Tipo

Un Arke è la *definizione* di qualcosa. Descrive la forma che un'informazione deve avere: quali campi contiene, di che tipo sono, con quali vincoli.

La caratteristica distintiva è che un Arke è esso stesso un dato nel sistema — non una classe nel codice, non una tabella hardcoded. Può essere creato, modificato, rimosso come qualsiasi altra informazione.

Questo è il cuore del meta-modello: **il modello è modellabile**.

---

### Unit — L'Istanza

Una Unit è la realizzazione concreta di un Arke. È l'occorrenza specifica che rispetta la struttura definita dal suo tipo.

Ogni Unit conosce il proprio Arke di appartenenza, e il sistema garantisce automaticamente che i dati siano coerenti con la definizione. Non serve scrivere validazione per ogni entità: la validazione è implicita nella struttura.

---

### Parameter — Il Campo

Un Arke è composto da Parameters. Ogni Parameter rappresenta una proprietà — testo, numero, data, booleano, riferimento ad altra entità, e molti altri tipi.

I Parameters sono riutilizzabili: lo stesso campo "data di creazione" o "nome" può essere condiviso tra più Arke senza essere ridefinito. Questo crea un vocabolario comune all'interno del sistema.

La validazione in Arke si articola su due livelli distinti:

**Sul Parameter**: ogni campo ha vincoli intrinseci legati al proprio tipo — una lunghezza massima per un testo, un intervallo per un numero, un formato per una data. Questi vincoli appartengono al Parameter stesso e si applicano ovunque esso venga usato.

**Sull'associazione Parameter–Arke**: quando un Parameter viene aggiunto a un Arke, è possibile specificare vincoli aggiuntivi che valgono solo in quel contesto. Lo stesso campo "descrizione" può essere facoltativo in un Arke e obbligatorio in un altro. Un valore di default può variare a seconda dell'Arke che lo utilizza.

Questa separazione permette di riutilizzare il vocabolario dei Parameters mantenendo il controllo granulare su come ogni campo si comporta nei diversi tipi di entità.

---

### Link — La Relazione

I Link esprimono le relazioni tra Unit. Un prodotto appartiene a una categoria, un utente è assegnato a un progetto, un documento ha un autore.

La particolarità dei Link in Arke è che sono anch'essi dati: hanno un tipo, possono portare metadati, e sono interrogabili come qualsiasi altra entità. Le relazioni non sono vincoli nascosti nello schema — sono connessioni esplicite e flessibili.

---

### Group — L'Aggregazione

Un Group raccoglie più Arke sotto un'unica categoria logica. Non cambia la struttura dei singoli Arke, ma permette di trattarli come un insieme coerente.

Il valore dei Group emerge quando si vuole interrogare entità eterogenee attraverso un'interfaccia comune. Un gruppo "Contenuti" potrebbe raccogliere articoli, video e documenti — diversi per struttura, ma accomunati da un'identità logica.

---

### Project — Il Contesto

Ogni dato in Arke appartiene a un Project. Il Project è l'unità di isolamento: strutture, istanze, relazioni e configurazioni di un Project non interferiscono mai con un altro.

Un'unica infrastruttura Arke può servire simultaneamente clienti completamente diversi, ciascuno con il proprio modello dati. Il multi-tenancy non è un'aggiunta — è parte del disegno originale.

---

## L'Ecosistema

Arke non è un monolite ma un insieme di moduli indipendenti e componibili. Ogni componente ha una responsabilità precisa e può essere adottato in modo selettivo.

---

### arke — Il Core

È il nucleo del sistema. Contiene la logica del meta-modello: come si definiscono gli Arke, come si creano le Unit, come si gestiscono i cicli di vita, come si attraversano le relazioni.

Il core non dipende da nessuna tecnologia di persistenza o di trasporto. È pura logica di dominio.

---

### arke-server — Il Layer API

Espone il sistema verso l'esterno attraverso un'interfaccia HTTP. Ogni Arke definito diventa automaticamente accessibile tramite endpoint standardizzati per la gestione delle sue Unit.

Non serve scrivere controller o route per ogni nuova entità: il server le deriva dalla struttura del sistema in modo dinamico.

---

### arke-auth — L'Identità

Gestisce autenticazione e autorizzazione. Si integra con il sistema di Arke e Project per garantire che ogni operazione avvenga nel contesto corretto, con i permessi appropriati.

*Approfondito nella sezione Feature.*

---

### arke-postgres — La Persistenza

Adatta il core di Arke a PostgreSQL. Traduce le operazioni del meta-modello in query concrete, gestisce la sincronizzazione tra la struttura in memoria e lo stato nel database.

*Approfondito nella sezione Persistenza.*

---

### arke-console — L'Interfaccia

Un'applicazione web che permette di interagire con il sistema in modo visuale. Dalla Console è possibile definire Arke, gestire Unit, esplorare relazioni, configurare permessi — senza toccare codice o API direttamente.

È lo strumento pensato per chi gestisce il dominio, non per chi costruisce il sistema.

---

## Come Si Tengono Insieme

Il flusso è lineare e prevedibile:

1. Si definisce un Arke con i suoi Parameters
2. Il server espone automaticamente gli endpoint per quel tipo
3. Si creano Unit conformi alla struttura
4. Le Unit possono essere collegate tramite Link e aggregate in Group
5. Tutto avviene nel contesto di un Project isolato

La separazione tra i moduli garantisce che ciascuno possa evolvere indipendentemente. Un cambio nella persistenza non tocca il core. Un nuovo tipo di autenticazione non cambia come si modellano i dati.

---

*Continua in: [02b — Topologia e Interrogazione](./02b_topologia.md)*
