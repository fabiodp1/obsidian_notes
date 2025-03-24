In [[Next.js]] è possibile creare endpoint API usando i [[route handler]]. Ovviamente se stiamo lavorando solo con [[Server Component]] il [[DB]] verrà contattato direttamente senza il layer dell'[[API]].

# Fetch dati nei [[Server Component]]

Di default le applicazioni [[Next.js]] utilizzano i [[React]] [[Server Component]]. Il fetch dei dati dai [[Server Component]] è relativamente un nuovo approccio e possiede i suoi vantaggi:

- Supportano le promises, fornendo una soluzione più semplice per le task asincrone. E' possibile usare `async/await` senza bisogno di [[useEffect]], [[useState]] o librerie per il fetch dei dati.
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

## Suspense & Streamed Responses

