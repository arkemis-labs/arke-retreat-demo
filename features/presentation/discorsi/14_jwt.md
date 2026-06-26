# Slide 14 — JWT

*Ritmo: Veloce. Testo da dire; le slide verranno adattate dopo.*

---

Sull'autenticazione sarò brevissimo, perché il concetto lo avete già: anche utenti e permessi sono Unit e Link come tutto il resto.

L'unica cosa concreta che vi dico: per le sessioni usiamo i JWT, tramite Guardian. Due token — un access token, che autentica ogni richiesta, e un refresh token, per rinnovarlo senza rifare il login. E dentro al token non viaggia un utente generico, ma il Member: l'identità già calata in uno specifico project, con il suo ruolo. È lui la "resource" di ogni richiesta autenticata.

Tutto qui. Andiamo alla parte interessante: i permessi.
