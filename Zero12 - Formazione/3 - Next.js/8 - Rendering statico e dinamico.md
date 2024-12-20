# Static rendering

## WHAT
Con lo [[static rendering]] il fetch dei dati e il [[rendering]] avviene server side a build-time o quando i dati vengono rivalidati.
## WHY
Ogni qualvolta un utente visita l'applicazione, viene servito il risultato cachato. Ci sono certi benefici:
- Siti web più veloci: il contenuto pre-renderizzato può essere cachato e distribuito globalmente. Questo assicura che gli utenti possano accedere al contenuto della pagina in maniera più veloce.
- Minore caricamento lato server: poichè il contenuto viene cachato, il server non deve generare dinamicamente il contenuto ad ogni richiesta.
- [[SEO]]
## WHEN
E' utile per UI senza dati o dati condivisi fra utenti, come i post di un blog statico o la pagina di un prodotto.
Non è una buona idea utilizzarlo per una dashboard con dati personalizzati che vengono aggiornati regolarmente.
# Dynamic rendering

## WHAT
Il [[dynamic rendering]] è l'opposto dello [[static rendering]], il contenuto viene renderizzato server-side ad ogni richiesta dell'utente.
## WHY
Il benefici sono:
- **Dati in real-time**: permette all'applicativo di mostrare in real-time dati aggiornati. Ideal in applicativi in cui sono frequenti i cambiamenti di dati.
- **Contenuto specifico per l'utente**: è più semplice servire contenuti personalizzati e aggiornarli all'interazione dell'utente.
- **Informazione alla richiesta**: il [[dynamic rendering]] permetto di accedere ad informazioni che possono essere note solo a request time, come cookies o i parametri di ricerca dell'[[URL]].
