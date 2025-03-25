# WHAT

A differenza di vanilla [React](React.md), Next oltre ai `client component` utilizza anche i `server component`.

>Di default tutti i tipi di componenti creati in [Next.js](Next.js), sono `server components`, generati ed eseguiti sul server (ad es. se mettiamo un console.log all'interno di un componente, sarà possibile vederlo solo sulla console del server, non del browser).

Questo fa in modo che ci sia meno codice JS lato client, rendendo l'app più veloce e migliorando la [SEO](SEO).

>Al contrario i `client components` sono componenti che vengono pre-renderizzati sul server, ma poi anche potenzialmente sul client.

Normalmente avremo bisogno di dei `client component` quando:

- abbiamo necessità di eseguire del codice JS sul client e quindi rendere la pagina interattiva;
- se dobbiamo usare dei React [hook](hook). Non è possibile in componenti eseguiti server-side;
- se dobbiamo reagire ad azioni dell'utente, ad es. usare gli eventi del [[DOM]], come click ecc.

Per poter rendere un componente client, basta usare la direttiva `"use client"` in cima al file:

```tsx
'use client'

import { ... } from '...';
//...
```

>All'interno di un `server component` è possibile fare il fetch diretto dei dati e utilizzare `async/await`.

---

# Usare i Client Components efficientemente

Quando decidiamo di rendere un componente `client`, dobbiamo ben considerare se è necessario che tutto il componente debba essere eseguito lato client, se possibile è molto meglio individuare e separare dei sottocomponenti che abbiano veramente necessità di essere eseguiti lato client, in modo da lasciare il resto server-side.

>Se è possibile includere componenti `client` in quelli `server`, non è possibile il contrario, SOLO SE inclusi come `children`:

```tsx
// YES <====

// ServerComponent.tsx
<h2>I'm a RSC!</h2>
<ClientComponent />

=================================
// NO <====

// ClientComponent.tsx
<h2>I'm a Client Cmp!</h2>
<ServerComponent />

=================================
// YES <====

// ClientComponent.tsx
<h2>I'm a Client Cmp!</h2>
{ children }

// page.tsx
<ClientComponent>
 <ServerComponent />
</ClientComponent>
```

---

# Rendering Types

## Static rendering

### WHAT

Con lo [[static rendering]] il fetch dei dati e il [[rendering]] avviene server side a build-time o quando i dati vengono rivalidati.
### WHY

Ogni qualvolta un utente visita l'applicazione, viene servito il risultato cachato. Ci sono certi benefici:
- Siti web più veloci: il contenuto pre-renderizzato può essere cachato e distribuito globalmente. Questo assicura che gli utenti possano accedere al contenuto della pagina in maniera più veloce.
- Minore caricamento lato server: poichè il contenuto viene cachato, il server non deve generare dinamicamente il contenuto ad ogni richiesta.
- [[SEO]]
### WHEN

E' utile per UI senza dati o dati condivisi fra utenti, come i post di un blog statico o la pagina di un prodotto.
Non è una buona idea utilizzarlo per una dashboard con dati personalizzati che vengono aggiornati regolarmente.

## Dynamic rendering

### WHAT

Il [[dynamic rendering]] è l'opposto dello [[static rendering]], il contenuto viene renderizzato server-side ad ogni richiesta dell'utente.
### WHY

Il benefici sono:

- **Dati in real-time**: permette all'applicativo di mostrare in real-time dati aggiornati. Ideal in applicativi in cui sono frequenti i cambiamenti di dati.
- **Contenuto specifico per l'utente**: è più semplice servire contenuti personalizzati e aggiornarli all'interazione dell'utente.
- **Informazione alla richiesta**: il [[dynamic rendering]] permetto di accedere ad informazioni che possono essere note solo a request time, come cookies o i parametri di ricerca dell'[[URL]].

