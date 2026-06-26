# Slide 10 — Managers ed ETS

*Ritmo: Veloce. Testo da dire; le slide verranno adattate dopo.*

Dobbiamo capire se tenerla in base a quello che diranno i filosofi

---

Ho nominato più volte "i manager". Due parole su cosa sono, veloce, perché la parte su ETS il collega l'ha già raccontata.

Le definizioni — gli Arke, i Parameter, i Group — sono i dati più letti del sistema: ogni operazione su una Unit deve controllarne la conformità col tipo. Leggerle dal database ogni volta sarebbe un collo di bottiglia. Quindi vivono in ETS, la memoria velocissima di cui vi ha parlato il collega, dietro dei registri: l'ArkeManager, il ParameterManager, il GroupManager.

Poi ci sono i manager operativi, quelli che lavorano davvero. Il QueryManager, che abbiamo appena visto. Il LinkManager, che muta la topologia — scrive e cancella le righe di `arke_link`. Lo StructManager, che traduce le Unit da e verso JSON: è lui che prepara le risposte dell'API. E il FileManager, una cache degli URL firmati dei file.

Non serve ricordarli tutti. Il messaggio è: dati caldi in memoria, e un manager dedicato per ogni responsabilità.
