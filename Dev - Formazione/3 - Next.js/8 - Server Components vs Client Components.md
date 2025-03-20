 A differenza di vanilla [React](React.md), Next oltre ai `client component` utilizza anche i `server component`.

>Di default tutti i tipi di componenti creati in [Next.js](Next.js), sono `server components`, generati ed eseguiti sul server (ad es. se mettiamo un console.log all'interno di un componente, sarà possibile vederlo solo sulla console del server, non del browser).

Questo fa in modo che ci sia meno codice JS lato client, rendendo l'app più veloce e migliorando la [SEO](SEO).

>Al contrario i `client components` sono componenti che vengono pre-renderizzati sul server, ma poi anche potenzialmente sul client.

Se abbiamo necessità di eseguire del codice JS sul client e quindi rendere la pagina interattiva, avremo bisogno di un `client component`, ad es.:

- se dobbiamo usare dei React [hook](hook), non possiamo in componenti renderizzati server-side;
- se dobbiamo reagire ad azioni dell'utente, ad es. usare gli eventi del [[DOM]], come click ecc.

Per poter rendere un componente client, basta usare la direttiva `"use client"` in cima al file:

```tsx
'use client'

import { ... } from '...';
//...
```

# Usare i Client Components efficientemente

Quando decidiamo di rendere un componente `client`, dobbiamo ben considerare se è necessario che tutto il componente debba essere eseguito lato client, se possibile è molto meglio individuare e separare dei sottocomponenti che hanno veramente necessità di essere eseguiti lato client, in modo da lasciare il resto server-side.