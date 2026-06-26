# Slide 9 — I limiti del QueryManager

*Ritmo: Veloce. Testo da dire; le slide verranno adattate dopo.*

---

Sarei disonesto se vi facessi vedere solo i pregi. Il QueryManager ha dei costi, ed è giusto conoscerli.

Primo: ritorna sempre l'intera Unit. Non potete chiedere solo due colonne.

Secondo: gli operatori sono un set predefinito — uguaglianza, LIKE, range, IN, IS NULL.

Terzo: ogni scrittura passa per i lifecycle hook. È potente, ma a volte è lento, e in certi scenari diventa un problema.

Quarto: non ci sono operazioni bulk. Insert, update e delete massivi, che in SQL puro sarebbero rapidissimi, qui non ci sono.

E quinto: per fare un update o un delete dovete prima fare una select, perché vi serve l'intera Unit in mano.

Quando questi limiti pesano davvero, c'è una via d'uscita: si scende a Ecto direttamente, con `pseudo_query`, per join custom, aggregati, window function — tutto ciò che gli operatori predefiniti non coprono. Ma è l'eccezione, non la regola.
