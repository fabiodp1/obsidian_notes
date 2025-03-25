In [[Next.js]] è possibile creare endpoint API usando i [[route handler]]. Ovviamente se stiamo lavorando solo con [[Server Component]] il [[DB]] verrà contattato direttamente senza il layer dell'[[API]].

# Fetch dati nei [[Server Component]]

Di default le applicazioni [[Next.js]] utilizzano i [[React]] [[Server Component]]. Il fetch dei dati dai [[Server Component]] è relativamente un nuovo approccio e possiede i suoi vantaggi:

- Supportano le `Promise`, fornendo una soluzione più semplice per le task asincrone. E' possibile usare `async/await` senza bisogno di [[useEffect]], [[useState]] o librerie per il fetch dei dati.
- Vengono eseguiti nel server, migliorando le performance. Il client riceverà direttamente il risultato.
- Essendo eseguiti nel server, è possibile contattare direttamente il [[DB]].

---

# Utilizzo di [[SQL]]

Per fare query verso il [[DB]] utilizzando [[Vercel Postgres]], importare la funzione `sql` da [[@vercel/postgres]].

```ts
import {sql} from '@vercel/postgres';

// ...
```

>In alternativa basta utilizzare il metodo messo a disposizione dalla libreria legata al tipo di [DB](DB) che stiamo utilizzando.

---

# Request waterfalls

Una [[request waterfall]] fa riferimento ad una sequenza di richieste network che dipendono dal completamento di quella precedente, ogni richiesta può iniziare solo quando la precedente ha restituito dati:

```tsx
const revenue = await fetchRevenue();
const latestInvoices = await fetchLatestInvoices(); // wait for fetchRevenue() to finish
const {
  numberOfInvoices,
  numberOfCustomers,
  totalPaidInvoices,
  totalPendingInvoices,
} = await fetchCardData(); // wait for fetchLatestInvoices() to finish
```

Nell'esempio bisogna aspettare che `fetchRevenue` venga eseguito prima che `fetchLatestInvoices` possa andare e così via.

>Questa situazione non è necessariamente cattiva, ci sono casi in cui vogliamo che ci sia una [[request waterfall]] perché vogliamo prima validare dei dati per decidere se proseguire con la chiamata successiva. Ma ci sono casi in cui non è intenzionale e può creare problemi di performance.

---

# Parallel data fetching

Un modo comune per evitare [[request waterfall]] è quello di inizializzare tutte le richieste allo stesso momento, in parallelo.

```ts
export async function fetchCardData() {
  try {
    const invoiceCountPromise = sql`SELECT COUNT(*) FROM invoices`;
    const customerCountPromise = sql`SELECT COUNT(*) FROM customers`;
    const invoiceStatusPromise = sql`SELECT
         SUM(CASE WHEN status = 'paid' THEN amount ELSE 0 END) AS "paid",
         SUM(CASE WHEN status = 'pending' THEN amount ELSE 0 END) AS "pending"
         FROM invoices`;
 
    const data = await Promise.all([
      invoiceCountPromise,
      customerCountPromise,
      invoiceStatusPromise,
    ]);
    // ...
  }
}
```

Utilizzando questo pattern è possibile:

- Iniziare i data fetch allo stesso tempo
- Utilizzare un pattern nativo JavaScript che può essere applicato a qualsiasi libreria o framework

>Esiste però uno svantaggio in questo pattern..cosa succede se una richiesta è più lenta di tutte le altre?

---

# Loading state

Nel momento in cui chiediamo una pagina che viene renderizzata lato server e che deve fare il fetch dei dati, essendo una operazione asincrona dovremo aspettare un tot tempo prima che sia pronta per essere servita, quindi ci serve un modo per gestire il feedback lato client.

Per questo come si fa di solito risulta utile avere un elemento di `loading`.
In [Next.js](Next.js) la cosa viene gestita automaticamente, basta aggiungere alla cartella `/app` il file speciale `loading.tsx`.

>Il componente ritornato da questo file verrà mostrato ogni volta che una pagina "fratello" o figlio caricherà dei dati.

```tsx title=loading.tsx
export default function MealsLoadingPage() {
  return <p>Fetching meals...</p>
}
```

## Partial Prerendering, Suspense & Streamed Responses

Il problema con l'utilizzo della pagina `loading.tsx` è che ogni qual volta viene caricato qualcosa, verrà mostrato il `loading` ma tutto il resto della pagina che sta facendo il fetch non verrà mostrato fino a quando il dato non sarà pronto, creando una [UX](UX) non ottimale.
la maggior parte delle [[route]] non sono completamente statiche o dinamiche. 
Ad es. il titolo della pagina ecc. potrebbero essere caricate subito e in seguito gli elementi dinamici, in modo da non dover mostrare tutto solo alla fine, quando i dati dinamici sono pronti.

>NB. Per la maggior parte delle web apps moderne, la scelta è fra [[static rendering]] e [[dynamic rendering]] per l'intera applicazione o per specifiche rotte. In [[Next.js]] se viene chiamata una [[dynamic function]] in una [[route]], l'intera [[route]] diventa dinamica.

[[PPR]] ([[partial prerendering]]) è un modello di rendering che permette di combinare i benefici del rendering statico e dinamico nella stessa [[route]]:

![[Pasted image 20241216113742.png]]

Quando un utente visita una rotta:

- Viene servito un guscio della rotta che include la navbar e le info sui prodotti, permettendo un caricamento iniziale più veloce.
- Il guscio lascia buchi in cui il contenuto del carrello e i prodotti suggeriti verranno caricati asincronicamente.
- I buchi asincroni vengono streammati in parallelo, riducendo il caricamento generale della pagina.

>**NB.** con questo pattern non sarà più necessario il componente `loading.tsx`.

Gli step da seguire per poter utilizzare questo pattern sono:

1. Estrapolare il contenuto dinamico, ad es. creando un apposito componente.
2. Fare il wrap di questo componente con `<Suspense>`.

Il [[partial prerendering]] utilizza `Suspense` e il contenuto dato alla prop `fallback` viene incluso nell'[[HTML]] iniziale con il contenuto statico, il contenuto statico così viene prerenderizzato creando il guscio statico. Il rendering del contenuto dinamico viene rimandato a quando l'utente richiederà la rotta.

>Wrappare un componente in `Suspense` non lo rende di per se dinamico, piuttosto `Suspense` viene utilizzato come confine fra il contenuto statico e quello dinamico.

```tsx title:page.tsx
// Parte dinamica
async function Meals() {
  const meals = await getMeals();
  return <MealsGrid meals={meals} />;
}

export default function MealsPage() {
  return (
    //...
    <main>
      // Confine fra contenuto statico e dinamico
      <Suspense fallback={<p>Fetching meals...</p>}>
        <Meals />
      </Suspense>
    </main>
    //...
  )
}
```

>Dietro le quinte è quello che viene fatto tramite il file `loading.tsx`, ma in questo caso lo stiamo limitando alla singola porzione di pagina dinamica, non tutta.

Per configurare la [PPR](PPR):

```ts
// next.config.ts
import type { NextConfig } from 'next';

const nextConfig: NextConfig = {
  experimental: {
    ppr: 'incremental'
  }
};
export default nextConfig;


// layout.tsx
// Può essere fatto anche a livello di singolo componente
import SideNav from '@/app/ui/dashboard/sidenav';

export const experimental_ppr = true; 

export default function Page() {
  return {
     <>
      <StaticComponent />
      <Suspense fallback={<Fallback />}>
        <DynamicComponent />
      </Suspense>
     </>
  };
}
```

>Non c'è bisogno di cambiare il codice, basta configurare e aver usato i `Suspense`.

# use() hook per Promise & Data Fetching

L'[hook](hook) `use()` può essere utilizzato per avere accesso al `Context` ma può essere anche utilizzato per fare l'`await` di `Promise` nel componenti `client`.
Lavora assieme a `Suspense` per gestire il data fetching e loading fallback.

>**IMPORTANTE** Non può essere usato con ogni tipo di `Promise`, ma con delle promise speciali, fornite dalla libreria legata a `Suspense`, oppure create all'interno di un `server component`.

Il problema è che quando facciamo l'`await` di una promise all'interno del `server component`, non avremo la pagina fino a quando non sarà risolta la promise, creando una [UX](UX) non delle migliori.
Per questo come abbiamo visto, è possibile fare il wrap del componente in questione, utilizzando `Suspense` e valorizzando la sua prop `fallback`.

Il problema si pone nel momento in cui vogliamo che questo componente sia `client` e quindi non sarà possibile usare l'await, visto che il componente non può più essere dichiarato come una funzione asincrona.
Dovremo spostare la logica asincrona nel `server component` padre e passare al figlio `client` la prop con il risultato della promise, ed è qui che l'[hook](hook) `use` diventa utile. Infatti considerando che il padre non dovrà utilizzare il risultato della `Promise` ma farà solo da ponte per il figlio, possiamo passare a quest'ultimo direttamente la `Promise`. Tramite `use` potremo aspettare che la promise venga risolta, ma `client-side`:

```tsx
// ServerComponent.tsx

export default function ServerComponent() {
  const userPromise = getUserFromDb();

  return (
    ...
    <ClientComponent userPromise={ userPromise } />
    ...
  );
}

// ClientComponent.tsx
"use client";
import { use } from 'react';

export default function ClientComponent({ userPromise }) {
  const user = use(userPromise); // await automatico
  ...
}

```