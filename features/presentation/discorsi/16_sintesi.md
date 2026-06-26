# Slide 16 — Sintesi: concetto → implementazione

*Ritmo: Veloce. Testo da dire; le slide verranno adattate dopo.*

---

Chiudiamo tornando da dove siamo partiti.

Il collega vi ha dato i concetti. Noi vi abbiamo fatto vedere che, sotto, c'è qualcosa di sorprendentemente semplice.

I tipi e le istanze? Righe di `arke_unit`. Le relazioni, i gruppi, i permessi? Righe di `arke_link`, distinte da una colonna `type`. La struttura del sistema? Dichiarata nel Registry, versionata in git. L'API? Un solo router dinamico, che serve ogni tipo senza che scriviate una rotta. La logica di dominio? Hook e funzioni custom. L'identità e la sicurezza? Ancora Unit e Link.

Tutto quello che avete sentito nella prima parte si riduce a questo: due tabelle, un registry, un router. È questo che intendiamo quando diciamo che in Arke il modello è un dato — e che è il sistema ad adattarsi al dominio, non il contrario.

Grazie. Domande?
