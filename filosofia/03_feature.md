# Arke — Feature

---

## Oltre la Modellazione

Definire strutture dati ed esporre API è il punto di partenza. Arke va oltre: fornisce un insieme di meccanismi che trasformano il meta-modello in un sistema operativo completo, senza richiedere implementazioni custom per ogni progetto.

---

## Autenticazione e Gestione dell'Identità

Arke integra nativamente la gestione dell'identità attraverso il modulo **arke-auth**.

L'autenticazione non è un componente esterno da collegare: è parte del sistema. Gli utenti sono anch'essi Unit — entità modellate come qualsiasi altro dato, con la differenza che il sistema li riconosce come soggetti delle operazioni, non solo oggetti.

Il risultato pratico:

- Registrazione, login e gestione delle sessioni sono disponibili senza configurazione aggiuntiva
- Il ciclo di vita dell'identità (creazione account, reset credenziali, revoca accessi) segue gli stessi meccanismi degli altri Arke
- L'integrazione con il sistema di permessi è automatica: ogni operazione avviene nel contesto dell'utente che la esegue

---

## Visibilità e Permessi

Il controllo degli accessi in Arke non è un layer separato — è integrato nella struttura stessa del sistema.

I permessi si articolano su più livelli:

**A livello di Project**: chi può accedere a un determinato contesto e con quale ruolo.

**A livello di Arke**: quali tipi di dato un utente può leggere, creare, modificare o eliminare.

**A livello di Unit**: visibilità granulare sulle singole istanze — un'entità può essere visibile a tutti, solo al suo creatore, o a un gruppo specifico di utenti.

Questo modello permette di costruire sistemi con logiche di accesso complesse senza scrivere codice di autorizzazione ad hoc. Le regole di visibilità sono configurazione, non implementazione.

---

## Hook sul Ciclo di Vita

Ogni Arke può reagire agli eventi del proprio ciclo di vita. Arke definisce quattro momenti intercettabili:

- **Prima della creazione**: validazioni aggiuntive, trasformazioni dei dati in ingresso, arricchimento da sorgenti esterne
- **Dopo la creazione**: notifiche, propagazioni, effetti collaterali
- **Prima dell'aggiornamento**: controlli di coerenza, blocco di modifiche non consentite
- **Prima dell'eliminazione**: pulizia di relazioni, archiviazione, vincoli di integrità custom

Gli hook non sostituiscono la validazione strutturale — quella è garantita dalla definizione del tipo. Gli hook gestiscono la *logica di dominio* che va oltre la struttura: le regole di business specifiche di ogni contesto.

Il vantaggio è che questa logica rimane localizzata nell'Arke che la riguarda. Non si disperde in middleware generici o listener globali.

---

## Funzioni Custom

Oltre agli hook, Arke permette di definire **funzioni custom** accessibili come endpoint dedicati su un Arke specifico.

Dove gli hook reagiscono a eventi standard (crea, aggiorna, elimina), le funzioni custom introducono operazioni nuove: un Arke "Ordine" potrebbe esporre una funzione "conferma", un Arke "Documento" potrebbe esporre "pubblica" o "archivia".

Queste funzioni:

- Sono associate all'Arke che le definisce, non al sistema in generale
- Sono accessibili tramite API con lo stesso meccanismo degli endpoint standard
- Possono orchestrare operazioni complesse su più entità mantenendo la coerenza transazionale

Il risultato è un sistema che supporta non solo il CRUD generico, ma il vocabolario specifico di ogni dominio.

---

## Composizione delle Feature

Queste capacità non sono silos separati — si compongono:

Un hook di creazione può verificare i permessi dell'utente corrente prima di procedere. Una funzione custom può essere protetta da un livello di visibilità specifico. Un cambio di stato orchestrato da una funzione può generare notifiche gestite da un hook.

La coerenza del modello garantisce che queste composizioni siano prevedibili e controllabili.

---

*Continua in: [04 — Persistenza](./04_persistenza.md)*
