# Arke — Feature

---

## Oltre la Modellazione

Definire strutture dati ed esporre API è il punto di partenza. Arke va oltre: fornisce un insieme di meccanismi che trasformano il meta-modello in un sistema operativo completo, senza richiedere implementazioni custom per ogni progetto.

---

## Autenticazione e Gestione dell'Identità

Arke integra nativamente la gestione dell'identità attraverso il modulo **arke-auth**.

L'autenticazione non è un componente esterno da collegare: è parte del sistema. Gli utenti sono anch'essi Unit — entità modellate come qualsiasi altro dato -, con la differenza che il sistema li riconosce come soggetti delle operazioni, non solo oggetti.

Il risultato pratico:

- Registrazione, login e gestione delle sessioni sono disponibili senza configurazione aggiuntiva
- Il ciclo di vita dell'identità (creazione account, reset credenziali, revoca accessi) segue gli stessi meccanismi degli altri Arke
- L'integrazione con il sistema di permessi è automatica: ogni operazione avviene nel contesto dell'utente che la esegue

---

## Visibilità e Permessi

Il controllo degli accessi in Arke non è un layer separato, ma è integrato nella struttura stessa del sistema.

I permessi si articolano su più livelli:

**A livello di Project**: chi può accedere a un determinato contesto e con quale ruolo.

**A livello di Arke**: quali tipi di dato un utente può leggere, creare, modificare o eliminare.

**A livello di Unit**: visibilità granulare sulle singole istanze: un'entità può essere visibile a tutti, solo al suo creatore, a un gruppo specifico di utenti o condizionata da un parametro.

Questo modello permette di costruire sistemi con logiche di accesso complesse senza scrivere codice di autorizzazione ad hoc. Le regole di visibilità sono configurazione, non implementazione.

---

## Hook sul Ciclo di Vita

Arke fornisce degli Hook, punti in cui lo sviluppatore può inserirsi per automatizzare delle operazioni durante il ciclo di vita della Unit.
Tutte le volte che viene creata, modificata, eliminata o interrogata una Unit lo sviluppatore può ovveridare dei comportanti tramite dei trigger. Questo comporta che tutte le operazioni verranno fatte in qualsiasi momento di modifica di quella Unit, il ché può comportare eventuali compromissioni dell'operatività. 
Usare con attenzione.

---

## Funzioni Custom

Oltre agli hook, Arke permette di definire **funzioni custom** accessibili come endpoint dedicati su un Arke specifico.

Dove gli hook reagiscono a eventi standard (crea, aggiorna, elimina), le funzioni custom introducono operazioni nuove.
Queste funzioni:

- Sono associate all'Arke che le definisce, non al sistema in generale
- Sono accessibili tramite API con lo stesso meccanismo degli endpoint standard
- Possono orchestrare operazioni complesse su più entità mantenendo la coerenza transazionale

Il risultato è un sistema che supporta non solo il CRUD generico, ma il vocabolario specifico di ogni dominio.

---

*Continua in: [04 — Persistenza](./04_persistenza.md)*
