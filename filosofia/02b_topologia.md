# Arke — Topologia e Interrogazione

---

## Due Modi di Leggere i Dati

In un sistema tradizionale c'è sostanzialmente un solo modo di interrogare i dati: si sceglie un tipo, si filtrano le istanze per attributo, si ottiene una lista. È il modello relazionale classico — potente, familiare, ma fondamentalmente piatto.

Arke supporta questo modello, ma lo affianca a un secondo approccio: **la traversata del grafo**.

Invece di partire da un tipo e filtrare, si parte da una Unit specifica e si naviga attraverso i suoi Link — seguendo le connessioni verso altre entità, che a loro volta possono portare verso altre ancora. Il dato non è più una riga in una tabella: è un nodo in una rete.

Questi due modi di leggere i dati non si escludono. Si scelgono in base alla domanda che si sta ponendo.

> "Tutti gli ordini con stato 'aperto'" è una domanda relazionale.
> "Tutto ciò che è collegato a questo cliente" è una domanda topologica.

---

## Due Tipi di Relazione

In Arke esistono due meccanismi distinti per esprimere che due entità sono connesse. La differenza non è tecnica — è concettuale.

---

### Il Parameter di Tipo Link — La Relazione Strutturale

Un Arke può avere tra i suoi Parameters uno di tipo *link*: un campo che non contiene un valore primitivo, ma un riferimento a un'altra Unit.

Questa relazione è **parte della definizione del tipo**. È strutturale, dichiarata nello schema dell'Arke, visibile a chiunque guardi la sua forma.

Pensa a un Arke "Articolo" che ha un parameter "autore" che punta a un Arke "Utente". Ogni articolo porta con sé il riferimento al proprio autore. La relazione è 1:N per costruzione: un articolo ha un autore, non molti.

Il parameter di tipo link è il modo giusto per esprimere dipendenze che fanno parte dell'identità di un'entità — relazioni senza le quali l'entità stessa non avrebbe senso completo.

---

### Il Link Autonomo — La Relazione Topologica

Non tutte le relazioni appartengono alla struttura di uno dei due soggetti coinvolti. A volte la connessione esiste *tra* le entità, non *dentro* a una di esse.

Un utente può partecipare a molti progetti. Un progetto può coinvolgere molti utenti. Nessuno dei due "possiede" questa relazione: è una connessione che esiste nel mezzo, autonoma rispetto alla struttura di entrambi.

Questi sono i **Link autonomi**: connessioni definite direttamente in topologia, senza essere parte della definizione di nessuno dei due Arke coinvolti. Sono il meccanismo naturale per le relazioni **N:N**.

Il Link autonomo ha alcune caratteristiche distintive:

- **Non appartiene a nessun Arke**: non modifica la struttura dei soggetti che connette
- **Porta metadati propri**: la connessione stessa può avere attributi (es. il ruolo di un utente in un progetto, la data di inizio di una collaborazione)
- **È interrogabile come entità**: si può chiedere al sistema "chi è connesso a chi" senza passare per la struttura degli Arke coinvolti

---

## Direzione e Parent-Child

Ogni Link in Arke è **direzionato**: non connette due entità in modo simmetrico, ma va da una **Unit parent** a una **Unit child**.

Questa direzione non è un dettaglio tecnico — è parte del significato della relazione. "Categoria contiene Prodotto" e "Prodotto appartiene a Categoria" descrivono la stessa connessione letta da angolazioni opposte, ma la direzione del Link stabilisce quale lettura è quella canonica.

In pratica questo significa che ogni Link ha sempre un verso di percorrenza primario, e il sistema permette di navigarlo in entrambe le direzioni:

- **Dalla parte del parent**: "tutti i figli di questa Unit" — si percorre il Link nel suo verso naturale
- **Dalla parte del child**: "tutti i genitori di questa Unit" — si percorre il Link a ritroso

Questo vale per entrambi i tipi di relazione. Un parameter di tipo link stabilisce una direzione implicita nella struttura dell'Arke. Un Link autonomo ha una direzione esplicita dichiarata nel momento in cui viene creato.

La direzione è ciò che trasforma un insieme di connessioni in un **grafo orientato**: una struttura che si può attraversare con intenzione, seguendo il flusso o risalendolo.

---

## La Topologia come Vista del Sistema

Mettendo insieme i due tipi di relazione, emerge una **topologia**: una mappa delle connessioni tra le entità del sistema.

Questa topologia non è una visualizzazione secondaria o un'aggiunta estetica. È una rappresentazione equivalente e complementare al modello relazionale. Le stesse informazioni, viste da un'angolazione diversa.

In topologia, un sistema Arke si presenta come un grafo:

- I **nodi** sono le Unit
- Gli **archi** sono i Link — sia quelli strutturali (parameter di tipo link) che quelli autonomi — ognuno con la propria direzione
- Il **tipo** di ogni nodo è il suo Arke di appartenenza
- I **Group** aggregano famiglie di nodi con identità comune

Navigare questo grafo permette di rispondere a domande che il modello relazionale faticherebbe ad esprimere in modo diretto: catene di dipendenze, reti di collaborazione, gradi di separazione tra entità.

---

## Scegliere il Modello Giusto

| Domanda | Modello |
|---|---|
| "Tutte le Unit di un certo tipo con questi attributi" | Relazionale |
| "Cosa è connesso a questa Unit specifica" | Topologico |
| "Relazione che appartiene a un'entità" | Parameter di tipo link |
| "Relazione che esiste tra due entità senza appartenerle" | Link autonomo |
| "Connessioni N:N con attributi propri" | Link autonomo |
| "Dipendenza strutturale (1:N, obbligatoria)" | Parameter di tipo link |
| "Tutti i figli di questa Unit" | Traversata diretta (parent → child) |
| "Tutti i parent di questa Unit" | Traversata inversa (child → parent) |

La scelta non è esclusiva: un sistema Arke maturo usa entrambi i modelli, per le domande giuste.

---

*Continua in: [03 — Feature](./03_feature.md)*
