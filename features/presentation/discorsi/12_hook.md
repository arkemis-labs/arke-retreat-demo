# Slide 12 — Hook del ciclo di vita

*Ritmo: Veloce. Testo da dire; le slide verranno adattate dopo.*

---

Gli hook li avete già incontrati come concetto, quindi vado veloce sul concreto.

Un hook è una callback sul ciclo di vita di un Arke. Ce n'è uno per ogni momento: prima e dopo la validazione, prima e dopo la creazione, l'aggiornamento e la cancellazione.

La cosa da vedere è la sequenza. Quando il QueryManager crea una Unit, le cose succedono in un ordine preciso: carica, valida, esegue il `before_create`, persiste, ed esegue l'`on_create`. Prima e dopo la scrittura. Il "before" per arricchire o bloccare, l'"on" per gli effetti collaterali — mandare una mail, per dire.

C'è anche una variante a livello di Group: un hook che si applica a tutti gli Arke di un gruppo in un colpo solo. Il punto giusto per la logica condivisa da più tipi.

Un avvertimento, che il collega ha già fatto e che ripeto: gli hook scattano a ogni operazione su quella Unit. Sono potentissimi e, proprio per questo, vanno usati con attenzione — è facile rallentare tutto senza accorgersene.

TODO aggiungere aneddoto di loop infinito tra due on_update